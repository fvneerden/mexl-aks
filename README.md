# Intro Azure AKS
With Azure Kubernetes Services (AKS) Microsoft offers a managed Kubernetes platform to customers. It is a ready to use Kubernetes platform without having to spend a lot of time creating a cluster with all the neccasarry configuration burden like networking, cluster masters and os patching of the cluster nodes. It also comes with pre configured monitoring seamless integrating with Azure Monitor and Log Analytics. Of corse still offering flexibility to integrate your own tools. Furthermore it is the plain vanilla Kubernetes and as such is fully compatible with any existing tooling you might have running on any other standard Kubernetes platform.
https://azure.microsoft.com/en-us/services/kubernetes-service/

# Media Excel Encoding
[MediaExcel](http://mediaexcel.com/) is an encoding and transcoding vendor offering physical appliance and software based encoding solutions. MediaExcel has been partnering with Microsoft for many years and engaging in Azure media customer projects. They are also listed as [recommended and tested contribution encoder](https://docs.microsoft.com/en-us/azure/media-services/latest/recommended-on-premises-live-encoders#live-encoders-that-output-fragmented-mp4) for [Azure Media Services](https://azure.microsoft.com/en-us/services/media-services/) for fMP4. There has also work done by both MediaExcel and Microsoft to integrate SCTE-35 timed metadata from MediaExcel encoder to an Azure Media Services Origin supporting Server Side Ad Insertion (SSAI) workflows. 

# Azure DevOps pipeline to bring it all together
All the general benefits that come with containerized workloads all apply in particular case. For this particular PoC we created an automated deployment pipeline for easy testing and deployment. With a deployment YAML and Pipeline YAML we can easilly automate deployment, provisioning and scaling of a Media Excel Encoding container. Once DevOps pushes the deployment job onto AKS a container image is pulled from Azure Container Registry. Although container images can be bulky utilizing node side caching any additional container pull is greatly improved down to seconds. With help of Media Excel we created a YAML file container pre- and post container lifecycle logic that will add and remove a container from Media Excel's management portal. This offers easy single pane of glass management of multiple instances across multiple node types, clusters and regions.
This deployment pipeline offers full flexibility to provision certain multi tenant customers or job priority on specific node types. This unlockes the posibility to provision encoding jobs on GPU enabled nodes for maximum throughput or use cheaper generic nodes for low priority jobs.
https://azure.microsoft.com/en-us/services/devops/
https://azure.microsoft.com/en-us/services/container-registry/

![Flow](/aks-architecture-diagram.jpg)

1)	Azure Pipeline is triggered to deploy onto AKS cluster. In the YAML file there is a reference to the MediaExcel Container in Azure Container Registry
2)	AKS starts deployment and pulls container from Azure Container Registry
3)	During Container start custom PHP script is loaded and container is added to the HMS (Hero Management Service). And placed into the correct device pool and job
4)	Encoder loads source and (in this case) push 4K livestream into Azure Media Services
5)	Media Services packaged Livestream into multiple formats and apply DRM
6)	Azure CDN scales livestream

# YAML MEXL-DEV.YAML
This YAML file contains all settings for the deployment onto AKS. There are a few custom settings to replace and these are **{BETWEEN CURLY BRACKETS}**

# YAML PIPELINE.YAML
This file as pretty simple. In DevOps it will instruct the agent to copy the deployment mexl-dev.yaml to the artifacts so it will be available for a release. 
During a release a simple **"kubectl apply -f {mexl-dev.yaml}"** is used. 

# VM-Application-deployment-with-Azure-Policy

Mission critical workloads often need to deploy hardened images to secure their application by pre-installing applications. This works most of the cases but increase over head of managing and maintaining the latest upgraded applications and reimaging. Azure VM Application a resource type under Azure compute Gallery that provides the flexibility to manage and deploy application across virtual machines and scalesets. It decouples application installation from VM Base image so you can streamline updates, reduce image maintanance overheadto accelerate deploiyment cycle. This functionality is best suited for deploying high scale, low latency, AI, micro-services, security and complient workloads.

This is great from managing application deployment on image but how to enforce all virtual machines or scalesets have a particular application installed or ensure all deployments in a subscription or respurce group have deployment of a particular application/s are of the most recent version. This can be achieved in conjunction of using Azure policy to enforce application deployment during VM deployment. In this post I will walk thrughr step by step process
      1) Creating VM Application in Compute Gallery
      2) Apply VM Application during VM deployment
      3) Create Azure policy to enforce VM Application deployment post VM deployment

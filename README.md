# wsl2_docker_gatk
Recently (September 2021), I started studying genomic data analysis, an exciting and challenging field. I want to run some data analysis on my computer, so I've been reading how to do it in Windows because it is the OS that I've been using my whole life, and there is some excellent software that I rely on daily, which wouldn't be available on Linux. So I decided to document the steps I took setting up the environment and tools. I do not intend to explain the tools here. I am only reporting the exact commands I executed after reading a plethora of documentation.

Docker is a widely-used container technology that allows users to create an isolated copy (instance) of one or more applications, including their dependencies and deploy them on a host system. With Docker Desktop running on WSL2, users can leverage Linux workspaces and avoid having to maintain both Linux and Windows build scripts. Here, I’ll use a test dataset from [Desafio LBB-Mendelics 2021](https://github.com/mendelics/lbb-mendelics-2021) and implement a workflow to NGS analysis using GATK and other tools.

![docker](https://drive.google.com/uc?export=view&id=15lR2xpD6DmOUouBpHlR9YdyXZQOO_e2B)

I am following the main steps for germline single-sample data steps recommended by the Broad Institute.

![gatk](https://drive.google.com/uc?export=view&id=15le0E7qlkpycOdwpdPcnrRZXFDbiOZoT)

First we have to install docker -> [wsl2_docker_installation](wsl2_docker_installation.md)

Now we are ready to pull the images and start analyzing data -> [gatk_using_docker](gatk_using_docker.md)


## Setup your own Deep Learning development environment using Docker

set up your own DL environement based on docker containers 

## Index 
- 1. Introduction 
	* why Docker 
- 2. Install Docker 
- 2.bis. The fast track 
- 3. Install Jupyterhub, your development API 
- 4 : Installing a Keras kernel for Jupyter
- 5: Build a docker image
- 6 : Share a host directory with the container
- 7 : Run your container as a service. 


```

FROM centos

MAINTAINER Robin KIPS <robin.kips@gmail.com>

# install Python with Anaconda
RUN yum -y install wget-1.14 bzip2-1.0.6 gcc-c++-4.8.5  && \
	wget --quiet http://repo.continuum.io/archive/Anaconda3-4.1.1-Linux-x86_64.sh -O /tmp/Anaconda3-4.1.1-Linux-x86_64.sh && \
	chmod +x /tmp/Anaconda3-4.1.1-Linux-x86_64.sh && \
	/tmp/Anaconda3-4.1.1-Linux-x86_64.sh  -b -p /opt/anaconda && \
	rm -f /tmp/* 

	# install npm, nodejs & js dependencies
RUN curl --silent --location https://rpm.nodesource.com/setup_4.x | bash - && \
	yum install -y nodejs-4.6.0 && \
	npm install -g configurable-http-proxy@1.3.0 && \ 
	opt/anaconda/bin/pip install jupyterhub==0.6.1  && \ 
	opt/anaconda/bin/pip install --upgrade --ignore-installed  setuptools==28.3.0 && \ 
	opt/anaconda/bin/pip install --upgrade notebook==4.2.3 

# install keras kernel
RUN /opt/anaconda/bin/conda create --name keras_kernel python=3.5.2  -y && \
	source /opt/anaconda/bin/activate keras_kernel && \
	/opt/anaconda/bin/conda install  -y theano=0.8.2 && \
	/opt/anaconda/bin/conda install -y h5py=2.6.0 && \
	pip install keras==1.1.1 && \
	pip install flask==0.11.1 && \
	/opt/anaconda/bin/conda install -y scikit-learn=0.18.1 pandas=0.19.1 ipykernel=4.5.1&& matplotlib=1.5.3 pillow=3.4.1 && \
	source /opt/anaconda/bin/deactivate 

#set alias : 
RUN echo 'alias conda="/opt/anaconda/bin/conda"' >> ~/.bashrc && \
	echo 'alias activate="/opt/anaconda/bin/activate"' >> ~/.bashrc && \
	echo 'alias deactivate="/opt/anaconda/bin/deactivate"' >> ~/.bashrc && \
	echo 'alias python_keras="/opt/anaconda/envs/keras_kernel/bin/python3"' >> ~/.bashrc && \
	echo 'alias jupyterhub="/opt/anaconda/bin/jupyterhub"' >> ~/.bashrc 

#set ENV
ENV KERAS_BACKEND=theano
ENV PATH=/opt/anaconda/bin:$PATH

#spawn jupyter in /home/ dir for access shared directories.
RUN mkdir -p /srv/jupyterhub/ && \
    cd /srv/jupyterhub/ && \ 
    /opt/anaconda/bin/jupyterhub --generate-config && \
    echo "c.Spawner.notebook_dir = '/home/'" >> jupyterhub_config.py

WORKDIR /srv/jupyterhub/
EXPOSE 8000
 
#start jupyterhub 
CMD ["jupyterhub", "-f", "jupyterhub_config.py"  "--no-ssl"]
```

---
layout: post
title: Running Jupyter Notebook on Remote Server
tags: [ipython, jupyter-notebook-config]
bigimg: /img/posts/2018-06-11-running-jupyter-notebook-on-remote/jupyter.png
---

### What is Jupyter Notebook?
The Jupyter Notebook is an open-source web application that allows you to create and share documents that contain live code, equations, visualizations and narrative text. Uses include: data cleaning and transformation, numerical simulation, statistical modeling, data visualization, machine learning, and much more.

You can explore it [here](http://jupyter.org/). It comes bundled with [Anaconda](https://www.anaconda.com/downloads) or can be installed using pip. See [here](http://jupyter.org/install.html) for installation docs.

### Problem
Now, by default Notebook server runs on localhost(127.0.0.1) and on port 8888, This can be easily accessed on the local machine but the problem arise when we want to access this active server of the jupyter notebook on remote as there we won't be able to access the localhost of the server locally.

### Solution
A workaround can be modifying the configuration file of Jupyter notebook you can do that by following the instructions below, once you have ensured that jupyter notebook is installed successfully on the remote server -

1. Generate the configuration file for jupyter notebook.
```
	$ jupyter notebook --generate-config
```
you might have to pass the parameter as follows 
```
	$ jupyter notebook --allow-root --generate-config
```
It would generate an output mentioning the location of the generated configuration file. In my case it was the folowing
```
	Writing default config to: /root/.jupyter/jupyter_notebook_config.py
```
2. Move to the location mentioned in the above output. And open the configuration file in a editor. Then modify the follwing IP address configuration under the NotebookApp Configuration section. By default it listens on localhost i.e 127.0.0.1, modify it to 0.0.0.0

![config-file](/img/posts/2018-06-11-running-jupyter-notebook-on-remote/config-file.png)

3. Now, you need to know the actual port mapping of the port on which you would like to run jupyter notebook. For ex - If you want to run jupyter notebook on port 8080 which in my case maps to port 31xxx, then you can start the jupyter notebook server with your new configuration as 
```
	$ jupyter notebook --allow-root --no-browser --port=8080
``` 
If you don't pass the port then it will run at the port mentioned in the configuration file which by default is 8888. Also if no mapping of port occur then you can directly use the port at which the notebook server is running.

4. Finally to start using the notebook copy the link which is generated after you started the notebook server then change the IP which will be 0.0.0.0 to the IP of your remote server and the port to the port at which you started the server or to the mapped port in case it exists.
For ex - in my case it was - http://Remote-Server-IP:(PORT or Mapped PORT)/?token=[generated-token-string]

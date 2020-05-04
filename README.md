# 在nvidia-docker中搭建TensorFlow-gpu/jupyter環境
---
## Environment
- OS Environment: **Ubuntu 16.04**
- NVIDIA GPU: **GeForce GTX 1050 (Notebooks)**
- NVIDIA GPU driver: **390.25**
- CUDA toolkit version: **9.0**
- cuDNN SDK: **libcudnn7-dev_7.0.5.15-1+cuda9.0_amd64**
- Docker-CE版本: **18.03.1~ce**
- tensorflow-gpu: **tensorflow/tensorflow:1.12.0-gpu-py3**

## Check Version
- [TensorFlow](https://www.tensorflow.org/install/gpu?hl=zh_tw)
- [NVIDIA Driver Downloads](https://www.nvidia.com/download/index.aspx?lang=en-us)
- [CUDA toolkit version](https://docs.nvidia.com/deploy/cuda-compatibility/index.html#binary-compatibility__table-toolkit-driver)
- [tensorflow-gpu](https://www.tensorflow.org/install/source?hl=zh-cn#linux)

## Installation
- Docker-CE
- nvidia-docker v2.0
- TensorFlow

### Install Docker-CE
安裝步驟

	# 檢查是否安裝過docker(若有要先移除舊版本) 
	$ docker -v 
	# 更新apt的package目錄
	$ sudo apt-get update
	# 安裝docker package來源
    $ sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    # 加入docker GPG Key(官方金鑰)
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	# 添加docker到apt
    $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
	# 更新apt的package目錄
	$ sudo apt-get update
	# 下載docker_ce=<VERSION_STRING> 指定docker版本
	$ sudo apt-get install docker-ce=18.03.1~ce-0~ubuntu

測試是否成功

	# 確認版本
	$ sudo docker version(docker -v)
	# 執行image確認是否安裝成功
	$ sudo docker run hello-world

將使用者加入docker群組 

	$ sudo usermod -aG docker $USER

### Install nvidia-docker v2.0
安裝步驟

	$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
	$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
	$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
	$ sudo apt-get update
	$ sudo apt-get install -y nvidia-docker2 
	$ sudo pkill -SIGHUP dockerd 

測試是否成功

	# 確認版本
	$ sudo nvidia-docker version
	# 執行image確認是否安裝成功(下載官方的docker image，並使用nvidia-smi指令驗證在docker container中有抓到gpu資訊)
	$ sudo docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi 

**問題解決**: 確認nvidia是否有運行!
    
	$ sudo docker info | grep nvidia
	# [1]  出現Runtimes: nvidia runc才是正確!
	
    # [2]  若出現 Swap limit support的問題!
    $ sudo gredit /etc/default/grub
	# 新增 GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1" 進去
	$ sudo update-grub
	# 重開機	
    $ sudo reboot
	$ sudo docker info | grep nvidia

### Install TensorFlow
到[dockerhub](https://hub.docker.com/search?type=image)找尋適當的image
	
	# 下載image
    $ sudo docker pull tensorflow/tensorflow:1.12.0-gpu-py3
	
	# 運行container
	# 把docker內的port 8888 與外部實體主機的port 8888接通，port 6006 與外部實體主機的port 6006接通
	$ nvidia-docker run --name <CONTAINER_NAME>-it -p 6006:6006 -p 8888:8888 tensorflow/tensorflow:1.12.0-gpu-py3

	# 記下token值
	# 開啟瀏覽器輸入http://localhost:8888
    # 貼上token值即可使用Jupyter notebook

將container中的Jupyter notebook資料夾鏈結到實體資料夾位置

	$ nvidia-docker run --name <CONTAINER_NAME>-it -p 6006:6006 -p 8888:8888 -v  <PATH>:/home tensorflow/tensorflow:1.12.0-gpu-py3


常用指令

	# 查看鏡像目錄
	$ docker images
	
	# 啟動container
	$ docker start <CONTAINER_NAME>
	# 停止container
	$ docker stop <CONTAINER_NAME>

	# 查看目前執行中的container(ps = process state)
	$ docker ps
	# 查看所有container(包含已停止的)
	$ docker ps -a

	# 刪除container
	$ sudo docker rm <CONTAINER_NAME>
	# 執行成功會顯示被刪除的名字，再用 ps -a 查詢會發現之前的容器已經消失!

	# 預設 rm 指令只能刪除停止的容器，如果要強制刪除執行中的容器，就要使用 -f 參數，代表 force 強制
	$ sudo docker rm -f <CONTAINER_NAME>	

	# 進入該container的bash方式
	# [1] 利用<CONTAINER ID>
	$ docker ps
	$ docker exec -it <CONTAINER ID> /bin/bash
	# [2] 利用<CONTAINER_NAME>
	$ docker exec -it <CONTAINER_NAME> /bin/bash
	# 進入會在notebooks資料夾(Jupyter notebook預設資料夾)

	# 若忘記jupyter notebook值(密碼)或是不小心登出了
	# 在bash裡頭輸入，即可找到token
	$ jupyter notebook list
	# 退出bash方式
	$ exit / Ctrl+d

## Note

[1]  ports:

	# "8888:8888"   # For Jupyter
	# "6006:6006"   # For TensorBoard

[2]  新增另一個container運行Jupyter notebook

	# 避免port衝突!
	# 把docker內的port 8888 與外部實體主機的port 8887接通 (-p 8887:8888)
	# 記下token值
	# 開啟瀏覽器輸入http://localhost:8887
	# 貼上token值即可使用Jupyter notebook

[3]  運行TensorBoard

	# 撰寫一個test.py，使用FileWriter
	# tf.summary.FileWriter('/tmp/tensorflow/tfboard_Test', sess.graph)

	$ docker exec -it <CONTAINER_NAME> /bin/bash
	$ python test.py 
	$ tensorboard --logdir=/tmp/tensorflow/tfboard_Test
	# 開啟瀏覽器輸入http://localhost:6006

[4]  查看docker儲存位置

	# 預設docker儲存位置: Docker Root Dir: /var/lib/docker
	$ docker info
	
	# 查看container儲存位置
    $ sudo ls -l var/lib/docker/containers 
	

## Resources
- [TensorFlow官方網站](https://www.tensorflow.org/install/docker)
- [Docker官方網站](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- [Linux 安裝 nvidia-docker2](https://roy051023.github.io/2019/02/25/Ubuntu-Install-Nvidia-Docker2/)
- [快速搭建Tensorflow深度學習環境–Nvidia-docker](https://darren1231.pixnet.net/blog/post/349736695)
- [基於docker安裝tensorflow](https://juejin.im/post/5a8fea695188257a7450cb4c)
- [使用Docker來建置Deep Learning環境](https://medium.com/@minyuantseng/%E4%BD%BF%E7%94%A8docker%E4%BE%86%E5%BB%BA%E7%BD%AEdeep-learning%E7%92%B0%E5%A2%83-171d35632840)
- [NVIDIA Docker v2 安裝](https://ellis-wu.github.io/2018/03/02/nvidia-docker-installation/)
- [使用docker安装tensorflow](https://www.jianshu.com/p/478750c45e68)
- [使用 TensorBoard 視覺化呈現 TensorFlow 計算流程教學](https://blog.gtwang.org/programming/tensorboard-tensorflow-visualization-tutorial/)
- [Docker 常用指令與容器操作教學](https://blog.gtwang.org/linux/docker-commands-and-container-management-tutorial/)
- [docker鏡像儲存在哪裡](https://blog.csdn.net/qq_30764991/article/details/81873610)


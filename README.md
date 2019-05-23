本文是在基于官方knative的基础上进行安装，因为官方yaml中涉及太多的gcr.io镜像和配置文件中也包含gcr.io，便于国内用户进行研究学习而进行替换为可拉取镜像的yaml文件。其中主要是替换镜像的操作。

直接使用替换好的官方yaml文件请移步[knative-image](https://github.com/huisebug/knative-yaml)

记录如下操作

knativev0.6.0官方yaml文件[knative官方github](https://github.com/knative/)
[knative官方安装方式](https://knative.dev/docs/install/knative-with-any-k8s/)

##官方下载yaml地址
https://github.com/knative/serving/releases/download/v0.6.0/serving.yaml 
https://github.com/knative/build/releases/download/v0.6.0/build.yaml https://github.com/knative/eventing/releases/download/v0.6.0/release.yaml 
https://github.com/knative/serving/releases/download/v0.6.0/monitoring.yaml 
https://github.com/knative/eventing-sources/releases/download/v0.6.0/eventing-sources.yaml 
https://raw.githubusercontent.com/knative/serving/v0.6.0/third_party/config/build/clusterrole.yaml


#yaml文件下载py脚本
```shell
cat <<EOF | python -

import os

packyamllist=['serving:serving',
                       'build:build',
                       'eventing:release',
                       'serving:monitoring',
                       'eventing-sources:eventing-sources'
                       ]
yamlurl=['https://raw.githubusercontent.com/knative/serving/v0.6.0/third_party/config/build/clusterrole.yaml']
for urlfile in packyamllist:
  ls=urlfile.split(':')
  pack=ls[0]
  yamlname=ls[1]
  yamlurl.append("https://github.com/knative/%s/releases/download/v0.6.0/%s.yaml" % (pack, yamlname))
for url in yamlurl:
  os.system('wget %s' % url)
  
EOF
```

#抓取yaml中的所有镜像地址，建立image文件夹及生成Dockerfile

##build.yaml

###其中包含对应镜像列表
['gcr.io/knative-releases/github.com/knative/build/cmd/creds-init@sha256:101f537b53b895b28b84ac3c74ede7d250845e24c51c26516873d8ccb23168ce']
['gcr.io/knative-releases/github.com/knative/build/cmd/git-init@sha256:ce2c17308e9cb81992be153861c359a0c9e5f69c501a490633c8fe54ec992d53']
['gcr.io/cloud-builders/gcs-fetcher']
['gcr.io/knative-releases/github.com/knative/build/cmd/nop@sha256:50e2be042298f24800b9840a9aef831a5fe4d89d9a8edea5e0559cdedf32369d']
['gcr.io/knative-releases/github.com/knative/build/cmd/creds-init@sha256:101f537b53b895b28b84ac3c74ede7d250845e24c51c26516873d8ccb23168ce']
['gcr.io/knative-releases/github.com/knative/build/cmd/git-init@sha256:ce2c17308e9cb81992be153861c359a0c9e5f69c501a490633c8fe54ec992d53']
['gcr.io/knative-releases/github.com/knative/build/cmd/nop@sha256:50e2be042298f24800b9840a9aef831a5fe4d89d9a8edea5e0559cdedf32369d']
['gcr.io/knative-releases/github.com/knative/build/cmd/controller@sha256:6a762848a46786cb481f5870787133e0d5e15615f8d54a5ba50d86b8315a58eb']
['gcr.io/knative-releases/github.com/knative/build/cmd/webhook@sha256:8f0bbc50b63f368c9959acab87838c6986691c28d424847459f3526bf97f8a3e']

###py内容

```shell
cat <<EOF | python -

import os,re

pwd=os.getcwd()
with open('build.yaml', 'r') as myfile:
  reg='gcr.io/.*'
  imgre=re.compile(reg)
  for yamlline in myfile:
    myimage = re.findall(imgre, yamlline)
    if myimage != []:
      #print(myimage)
      branch=myimage[0].split('knative')[-1].split('@')[0].replace('/','-').lstrip('-')
      #print(branch)
      if os.path.isdir(r'%s/build/%s' % (pwd, branch)):
        pass
      else:
        os.makedirs(r'%s/build/%s' % (pwd, branch))
      os.system('echo %s > %s/build/%s/Dockerfile' % ('FROM'+' '+myimage[0], pwd, branch))

EOF
```

##clusterrole.yaml没有涉及到镜像

##eventing-sources.yaml

其中包含对应镜像列表
['gcr.io/knative-releases/github.com/knative/eventing-sources/cmd/github_receive_adapter@sha256:b5d6e12d16d16c6c42ae3d4325a1ef3a8a129dfc97740aa28000c0867edfc4ff']
['gcr.io/knative-releases/github.com/knative/eventing-sources/cmd/manager@sha256:99cf1f559f74ae97f271632697ed6e78a3fdd88a155632a57341b0dd6eab6581']

###py内容

```shell
cat <<EOF | python -

import os,re

pwd=os.getcwd()
with open('eventing-sources.yaml', 'r') as myfile:
  reg='gcr.io/.*'
  imgre=re.compile(reg)
  for yamlline in myfile:
    myimage = re.findall(imgre, yamlline)
    if myimage != []:
      #print(myimage)
      branch=myimage[0].split('knative')[-1].split('@')[0].replace('/','-').lstrip('-')
      #print(branch)
      if os.path.isdir(r'%s/eventing-sources/%s' % (pwd, branch)):
        pass
      else:
        os.makedirs(r'%s/eventing-sources/%s' % (pwd, branch))
      os.system('echo %s > %s/eventing-sources/%s/Dockerfile' % ('FROM'+' '+myimage[0], pwd, branch))

EOF
```

##monitoring.yaml

###其中包含对应镜像列表
['k8s.gcr.io/elasticsearch:v5.6.4']
['k8s.gcr.io/fluentd-elasticsearch:v2.0.4']
['k8s.gcr.io/addon-resizer:1.7']

###py内容

```shell
cat <<EOF | python -

import os,re

pwd=os.getcwd()
with open('monitoring.yaml', 'r') as myfile:
  reg='k8s.gcr.io/.*'
  imgre=re.compile(reg)
  for yamlline in myfile:
    myimage = re.findall(imgre, yamlline)
    if myimage != []:
      #print(myimage)
      branch=myimage[0].replace('/','-').lstrip('-').replace(':','-')
      #print(branch)
      if os.path.isdir(r'%s/monitoring/%s' % (pwd, branch)):
        pass
      else:
        os.makedirs(r'%s/monitoring/%s' % (pwd, branch))
      os.system('echo %s > %s/monitoring/%s/Dockerfile' % ('FROM'+' '+myimage[0], pwd, branch))

EOF
```

##release.yaml

###其中包含对应镜像列表
[' gcr.io/knative-releases/github.com/knative/eventing/cmd/broker/ingress@sha256:a0acbe69420a67bef520e86aceaa237bf540c15882701c96245a6c4e06413bf6']
[' gcr.io/knative-releases/github.com/knative/eventing/cmd/broker/filter@sha256:b4da7ce7b12aff2355066ed3237aadcf35df3b1c78db83cc538e6cffa564f208']
[' gcr.io/knative-releases/github.com/knative/eventing/cmd/controller@sha256:85c010633944c06f4c16253108c2338dba271971b2b5f2d877b8247fa19ff5cb']
[' gcr.io/knative-releases/github.com/knative/eventing/cmd/cronjob_receive_adapter@sha256:6bbb724d5a4dbaaead890ea51d5f84eb9514974a2d06e26c8753db59010987fb']
[' gcr.io/knative-releases/github.com/knative/eventing/cmd/apiserver_receive_adapter@sha256:7349f83eebe85a3eed7cdc4d442a935deab1ba0c42f34294f219f4ef17b59fec']
[' gcr.io/knative-releases/github.com/knative/eventing/cmd/sources_controller@sha256:aaa48f71a8db1b1dcf86c57d2dd72be1a65ed76d77f23a5abef4b2ad5c01c863']
[' gcr.io/knative-releases/github.com/knative/eventing/cmd/webhook@sha256:34a7cac96f8c809a7ce8ea0a86445204bbc6ac897525b876f53babb325f50bdc']
[' gcr.io/knative-releases/github.com/knative/eventing/cmd/in_memory/controller@sha256:496c19e81d9e7e40b3887c7c290304934f54f46c8a9186e800e314c014970c26']
[' gcr.io/knative-releases/github.com/knative/eventing/cmd/in_memory/dispatcher@sha256:897f03ed16e0000944da9ee0fdc971c43c8a494ff771c4e64d0573caf357c013']
['k8s.gcr.io/fluentd-elasticsearch:v2.0.4']

####注意：因为这个yaml文件存在gcr.io和k8s.gcr.io的镜像，所以需要两次正则筛选

###py内容

```shell
cat <<EOF | python -

import os,re

pwd=os.getcwd()
reglist=['\sgcr.io/.*', 'k8s.gcr.io/.*']
for reg in reglist:
  imgre=re.compile(reg)
  with open('release.yaml', 'r') as myfile:
    for yamlline in myfile:
      myimage = re.findall(imgre, yamlline)
      if myimage != []:
        #print(myimage)
        branch=myimage[0].split('knative')[-1].split('@')[0].replace('/','-').lstrip('-').replace(':','-')
        #print(branch)
        if os.path.isdir(r'%s/release/%s' % (pwd, branch)):
          pass
        else:
          os.makedirs(r'%s/release/%s' % (pwd, branch))
        os.system('echo %s > %s/release/%s/Dockerfile' % ('FROM'+' '+myimage[0], pwd, branch))

EOF
```

##serving.yaml

###其中包含对应镜像列表
[' gcr.io/knative-releases/github.com/knative/serving/cmd/queue@sha256:1e40c99ff5977daa2d69873fff604c6d09651af1f9ff15aadf8849b3ee77ab45']
[' gcr.io/knative-releases/github.com/knative/serving/cmd/activator@sha256:f553b6cb7599f2f71190ddc93024952e22f2f55e97a3f38519d4d622fc751651']
[' gcr.io/knative-releases/github.com/knative/serving/cmd/autoscaler@sha256:3a466eaf05cd505338163322331ee8634c601204250fa639360ae3524756acc3']
[' gcr.io/knative-releases/github.com/knative/serving/cmd/queue@sha256:1e40c99ff5977daa2d69873fff604c6d09651af1f9ff15aadf8849b3ee77ab45']
[' gcr.io/knative-releases/github.com/knative/serving/cmd/controller@sha256:8f402eab0ada038d3de2ad753a40f9f441715d08058d890537146bb0aba11c8e']
[' gcr.io/knative-releases/github.com/knative/serving/cmd/networking/certmanager@sha256:dc77db09a23103f64a554de4e01cfda7371cbb13bc0954c991bdc4141169257f']
[' gcr.io/knative-releases/github.com/knative/serving/cmd/networking/istio@sha256:55fe9eeacfc20d97d3cd4f80bfc8a9b95cff7b5c50121bda87f754da8f05e57b']
[' gcr.io/knative-releases/github.com/knative/serving/cmd/webhook@sha256:f0f98736bd4b55354f447f59183bf26b9be1ab01691b8b4aeee85caeb1166562']
['k8s.gcr.io/fluentd-elasticsearch:v2.0.4']
['gcr.io/knative-releases/github.com/knative/eventing-sources/cmd/github_receive_adapter@sha256:b5d6e12d16d16c6c42ae3d4325a1ef3a8a129dfc97740aa28000c0867edfc4ff']
['gcr.io/knative-releases/github.com/knative/eventing-sources/cmd/manager@sha256:99cf1f559f74ae97f271632697ed6e78a3fdd88a155632a57341b0dd6eab6581']
['k8s.gcr.io/elasticsearch:v5.6.4']
['k8s.gcr.io/fluentd-elasticsearch:v2.0.4']
['k8s.gcr.io/addon-resizer:1.7']

###py内容

```shell
cat <<EOF | python -

import os,re

pwd=os.getcwd()
reglist=['\sgcr.io/.*', 'k8s.gcr.io/.*']
for reg in reglist:
  imgre=re.compile(reg)
  with open('serving.yaml', 'r') as myfile:
    for yamlline in myfile:
      myimage = re.findall(imgre, yamlline)
      if myimage != []:
        #print(myimage)
        branch=myimage[0].split('knative')[-1].split('@')[0].replace('/','-').lstrip('-').replace(':','-')
        #print(branch)
        if os.path.isdir(r'%s/serving/%s' % (pwd, branch)):
          pass
        else:
          os.makedirs(r'%s/serving/%s' % (pwd, branch))
        os.system('echo %s > %s/serving/%s/Dockerfile' % ('FROM'+' '+myimage[0], pwd, branch))

EOF
```

#设置git ssh登录并将各image的Dockerfile推送到不同的分支

##生成sshkey
```shell
ssh-keygen -t rsa -C "huisebug@outlook.com"
```

##shell内容

```shell
cat >gitpush.sh << EOF

#!/bin/bash
git config --global push.default current;
git config --global user.name "root" 
git config --global user.email "huisebug@outlook.com"
WORKDIR=/root/knative
for yamldir in build eventing-sources monitoring release serving
do
cd $WORKDIR/$yamldir;
for dir in $(ls -d */)
do
#去除"/"符号
branch=$(echo $dir | tr -d '/')
cd $WORKDIR/$yamldir/$dir;
git init;
git checkout -b $branch
git add -A Dockerfile;
git commit -m "commit $branch";
git remote add origin git@github.com:huisebug/knative-images.git;
git push -f -u origin $branch;
done
done

EOF
```

#所有分支信息，共27个镜像在gcr.io,其中的quay.io国内是可以拉取到的。
```shell
build-cmd-controller
build-cmd-creds-init
build-cmd-git-init
build-cmd-nop
build-cmd-webhook
gcr.io-cloud-builders-gcs-fetcher
k8s.gcr.io-addon-resizer-1.7
k8s.gcr.io-elasticsearch-v5.6.4
k8s.gcr.io-fluentd-elasticsearch-v2.0.4
eventing-sources-cmd-github_receive_adapter
eventing-sources-cmd-manager
eventing-cmd-apiserver_receive_adapter
eventing-cmd-broker-filter
eventing-cmd-broker-ingress
eventing-cmd-controller
eventing-cmd-cronjob_receive_adapter
eventing-cmd-in_memory-controller
eventing-cmd-in_memory-dispatcher
eventing-cmd-sources_controller
eventing-cmd-webhook
serving-cmd-activator
serving-cmd-autoscaler
serving-cmd-controller
serving-cmd-networking-certmanager
serving-cmd-networking-istio
serving-cmd-queue
serving-cmd-webhook
```

#分支对应在gcr.io的镜像，其中quay.io的镜像国内可以拉取，这也是image.txt

```shell
cat > image.txt << EOF
build-cmd-git-init
gcr.io/knative-releases/github.com/knative/build/cmd/git-init@sha256:ce2c17308e9cb81992be153861c359a0c9e5f69c501a490633c8fe54ec992d53
build-cmd-creds-init
gcr.io/knative-releases/github.com/knative/build/cmd/creds-init@sha256:101f537b53b895b28b84ac3c74ede7d250845e24c51c26516873d8ccb23168ce
build-cmd-nop
gcr.io/knative-releases/github.com/knative/build/cmd/nop@sha256:50e2be042298f24800b9840a9aef831a5fe4d89d9a8edea5e0559cdedf32369d
build-cmd-controller
gcr.io/knative-releases/github.com/knative/build/cmd/controller@sha256:6a762848a46786cb481f5870787133e0d5e15615f8d54a5ba50d86b8315a58eb
build-cmd-webhook
gcr.io/knative-releases/github.com/knative/build/cmd/webhook@sha256:8f0bbc50b63f368c9959acab87838c6986691c28d424847459f3526bf97f8a3e
gcr.io-cloud-builders-gcs-fetcher
gcr.io/cloud-builders/gcs-fetcher
eventing-cmd-broker-ingress
gcr.io/knative-releases/github.com/knative/eventing/cmd/broker/ingress@sha256:a0acbe69420a67bef520e86aceaa237bf540c15882701c96245a6c4e06413bf6
eventing-cmd-broker-filter
gcr.io/knative-releases/github.com/knative/eventing/cmd/broker/filter@sha256:b4da7ce7b12aff2355066ed3237aadcf35df3b1c78db83cc538e6cffa564f208
eventing-cmd-controller
gcr.io/knative-releases/github.com/knative/eventing/cmd/controller@sha256:85c010633944c06f4c16253108c2338dba271971b2b5f2d877b8247fa19ff5cb
eventing-cmd-cronjob_receive_adapter
gcr.io/knative-releases/github.com/knative/eventing/cmd/cronjob_receive_adapter@sha256:6bbb724d5a4dbaaead890ea51d5f84eb9514974a2d06e26c8753db59010987fb
eventing-cmd-apiserver_receive_adapter
gcr.io/knative-releases/github.com/knative/eventing/cmd/apiserver_receive_adapter@sha256:7349f83eebe85a3eed7cdc4d442a935deab1ba0c42f34294f219f4ef17b59fec
eventing-cmd-sources_controller
gcr.io/knative-releases/github.com/knative/eventing/cmd/sources_controller@sha256:aaa48f71a8db1b1dcf86c57d2dd72be1a65ed76d77f23a5abef4b2ad5c01c863
eventing-cmd-webhook
gcr.io/knative-releases/github.com/knative/eventing/cmd/webhook@sha256:34a7cac96f8c809a7ce8ea0a86445204bbc6ac897525b876f53babb325f50bdc
eventing-cmd-in_memory-controller
gcr.io/knative-releases/github.com/knative/eventing/cmd/in_memory/controller@sha256:496c19e81d9e7e40b3887c7c290304934f54f46c8a9186e800e314c014970c26
eventing-cmd-in_memory-dispatcher
gcr.io/knative-releases/github.com/knative/eventing/cmd/in_memory/dispatcher@sha256:897f03ed16e0000944da9ee0fdc971c43c8a494ff771c4e64d0573caf357c013
serving-cmd-queue
gcr.io/knative-releases/github.com/knative/serving/cmd/queue@sha256:1e40c99ff5977daa2d69873fff604c6d09651af1f9ff15aadf8849b3ee77ab45
serving-cmd-activator
gcr.io/knative-releases/github.com/knative/serving/cmd/activator@sha256:f553b6cb7599f2f71190ddc93024952e22f2f55e97a3f38519d4d622fc751651
serving-cmd-autoscaler
gcr.io/knative-releases/github.com/knative/serving/cmd/autoscaler@sha256:3a466eaf05cd505338163322331ee8634c601204250fa639360ae3524756acc3
serving-cmd-controller
gcr.io/knative-releases/github.com/knative/serving/cmd/controller@sha256:8f402eab0ada038d3de2ad753a40f9f441715d08058d890537146bb0aba11c8e
serving-cmd-networking-certmanager
gcr.io/knative-releases/github.com/knative/serving/cmd/networking/certmanager@sha256:dc77db09a23103f64a554de4e01cfda7371cbb13bc0954c991bdc4141169257f
serving-cmd-networking-istio
gcr.io/knative-releases/github.com/knative/serving/cmd/networking/istio@sha256:55fe9eeacfc20d97d3cd4f80bfc8a9b95cff7b5c50121bda87f754da8f05e57b
serving-cmd-webhook
gcr.io/knative-releases/github.com/knative/serving/cmd/webhook@sha256:f0f98736bd4b55354f447f59183bf26b9be1ab01691b8b4aeee85caeb1166562
eventing-sources-cmd-github_receive_adapter
gcr.io/knative-releases/github.com/knative/eventing-sources/cmd/github_receive_adapter@sha256:b5d6e12d16d16c6c42ae3d4325a1ef3a8a129dfc97740aa28000c0867edfc4ff
eventing-sources-cmd-manager
gcr.io/knative-releases/github.com/knative/eventing-sources/cmd/manager@sha256:99cf1f559f74ae97f271632697ed6e78a3fdd88a155632a57341b0dd6eab6581
k8s.gcr.io-addon-resizer-1.7
k8s.gcr.io/addon-resizer:1.7
k8s.gcr.io-elasticsearch-v5.6.4
k8s.gcr.io/elasticsearch:v5.6.4
k8s.gcr.io-fluentd-elasticsearch-v2.0.4
k8s.gcr.io/fluentd-elasticsearch:v2.0.4

EOF
```

中间建立github与dockerhub进行自动化构建

最后的操作即将5个yaml文件中的gcr.io镜像替换为我在dockerhub的镜像即可

##yaml文件替换shell脚本

```shell
cat >sedyaml.sh << EOF

#!/bin/bash
WORKDIR=/root/knative
cd $WORKDIR;

for ((i=1;i<54;i=i+2))
do
newimagetag=$(cat image.txt | sed -n "${i}p")
a=$(($i+1))
oldimage=$(cat image.txt  | sed -n "${a}p")

for yaml in $(ls *.yaml)
do
  sed -i  's#'${oldimage}'#huisebug/knative:'${newimagetag}'#g' ${yaml}
fi
done
done

EOF
```

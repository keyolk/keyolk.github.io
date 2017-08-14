+++
title  = "gcloud"
toc    = true
weight = 4
+++

## Intro

```bash
$ curl https://sdk.cloud.google.com | bash
```

```bash
exec -l $SHELL
```

```bash
$ gcloud auth login
```

## GKE

```bash
$ gcloud components update kubectl
```

```bash
$ gcloud config set project keyolk
$ glcoud config set compute/zone asia-east1-a
```

```bash
$ gcloud config set container/cluster keyolk
```

```bash
$ gcloud config list
```

```bash
$ gcloud container clusters create keyolk --num-nodes 2 --machine-type g1-small
```

```bash
$ gcloud compute instances list
```

```bash
$ kubectl run tomcat --image=tomcat
```

```bash
$ gcloud container clusters delete keyolk
```

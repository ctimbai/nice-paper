Docker 项目刚刚兴起时，Google 也开源了一个在内部使用多年、经历过生产环境验证的 Linux 容器：lmctfy（Let Me Container That For You）。

然而，面对 Docker 项目的强势崛起，这个对用户没那么友好的 Google 容器项目根本没有招架之力。所以，知难而退的 Google 公司，向 Docker 公司表示了合作的愿望：关停这个项目，和 Docker 公司共同推进一个中立的容器运行时（container runtime）库作为 Docker 项目的核心依赖。不过，Docker 公司并没有认同这个明显会削弱自己地位的提议，还在不久后，自己发布了一个容器运行时库 Libcontainer。这次匆忙的、由一家主导的、并带有战略性考量的重构，成了 Libcontainer 被社区长期诟病代码可读性差、可维护性不强的一个重要原因。至此，Docker 公司在容器运行时层面上的强硬态度，以及 Docker 项目在高速迭代中表现出来的不稳定和频繁变更的问题，开始让社区叫苦不迭。

于是，2015 年 6 月 22 日，由 Docker 公司牵头，CoreOS、Google、RedHat 等公司共同宣布，Docker 公司将 Libcontainer 捐出，并改名为 RunC 项目，交由一个完全中立的基金会管理，然后以 RunC 为依据，大家共同制定一套容器和镜像的标准和规范。这套标准和规范，就是 OCI（ Open Container Initiative ）。OCI 的提出，意在将容器运行时和镜像的实现从 Docker 项目中完全剥离出来。这样做，一方面可以改善 Docker 公司在容器技术上一家独大的现状，另一方面也为其他玩家不依赖于 Docker 项目构建各自的平台层能力提供了可能。

从 2017 年开始，Docker 公司先是将 Docker 项目的容器运行时部分 Containerd 捐赠给 CNCF 社区，标志着 Docker 项目已经全面升级成为一个 PaaS 平台；紧接着，Docker 公司宣布将 Docker 项目改名为 Moby，然后交给社区自行维护，而 Docker 公司的商业产品将占有 Docker 这个注册商标。

2017 年 10 月，Docker 公司出人意料地宣布，将在自己的主打产品 Docker 企业版中内置 Kubernetes 项目，这标志着持续了近两年之久的“编排之争”至此落下帷幕。

docker - containerd - runc - OCI格式的容器

libcontainer是containerd的前身，现在不提了

moby是继承了原先的docker的项目，是社区维护的的开源项目，谁都可以在moby的基础打造自己的容器产品

2017年年初，docker公司将原先的docker项目改名为moby，并创建了docker-ce和docker-ee。

docker-ce是docker公司维护的开源项目，是一个基于moby项目的免费的容器产品
docker-ee是docker公司维护的闭源产品，是docker公司的商业产品。moby是继承了原先的docker的项目，是社区维护的的开源项目，谁都可以在moby的基础打造自己的容器产品


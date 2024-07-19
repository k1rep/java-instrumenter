FROM openjdk:8-jdk-slim

# 使用可靠的Debian包源，并注释掉安全更新源
RUN sed -i 's|http://deb.debian.org/debian|http://mirrors.aliyun.com/debian|g' /etc/apt/sources.list && \
    sed -i 's|deb http://security.debian.org/debian-security|#deb http://security.debian.org/debian-security|g' /etc/apt/sources.list

# 更新包列表并安装必要的软件包
RUN apt-get update && \
    apt-get install -y software-properties-common && \
    apt-get update && \
    apt-get install -y \
        maven \
        git \
        build-essential \
        subversion \
        perl \
        curl \
        unzip \
        cpanminus \
        make && \
    rm -rf /var/lib/apt/lists/*

# 设置时区
ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 克隆Defects4j仓库
WORKDIR /
RUN git clone https://github.com/Frenkymd/defects4j.git /defects4j

WORKDIR /defects4j
RUN git checkout chain

# 使用不同的CPAN镜像源，并添加重试机制
RUN cpanm --mirror http://mirrors.163.com/cpan/ --mirror-only --installdeps . || \
    cpanm --mirror http://cpan.metacpan.org/ --mirror-only --installdeps .

# 下载并设置Defects4j的init.sh脚本内容
RUN curl -O https://defects4j.org/downloads/defects4j-repos.zip && \
    unzip defects4j-repos.zip -d /defects4j && \
    rm defects4j-repos.zip

# 分步骤执行init.sh中的各个部分以处理下载失败的问题
RUN curl -L -o major-1.3.4_jre7.zip https://mutation-testing.org/downloads/major-1.3.4_jre7.zip && \
    unzip major-1.3.4_jre7.zip -d /defects4j && \
    rm major-1.3.4_jre7.zip

RUN curl -L -o evosuite-1.1.0.jar https://github.com/EvoSuite/evosuite/releases/download/v1.1.0/evosuite-1.1.0.jar && \
    mv evosuite-1.1.0.jar /defects4j/framework/lib/

# 设置环境变量
ENV PATH="/defects4j/framework/bin:${PATH}"

# 构建项目
WORKDIR /instrumenter

COPY src src
COPY pom.xml .

# 复制本地库
COPY libs/fault-localization-0.0.1.jar /app/libs/fault-localization-0.0.1.jar

# 运行Maven命令并指定本地库目录
RUN mvn install:install-file -Dfile=/app/libs/fault-localization-0.0.1.jar -DgroupId=hu.szte.inf.sed -DartifactId=fault-localization -Dversion=0.0.1 -Dpackaging=jar

RUN mvn clean package

RUN cp target/method-agent-*-jar-with-dependencies.jar /defects4j/framework/lib/agent.jar

# 设置默认命令
CMD defects4j compile && defects4j test
FROM amazonlinux:2023



# 0. Instalar Docker explícitamente
# RUN dnf install -y docker
# 1. Actualizar e instalar herramientas básicas
RUN dnf update -y && \
    dnf install -y --allowerasing git unzip curl wget tar gzip jq findutils && \
    dnf clean all

# 2. Instalar Node.js 18
RUN curl -sL https://rpm.nodesource.com/setup_18.x | bash - && \
    dnf install -y nodejs

# 3. Configurar Python 3.9 + pip (sin actualizar el pip del sistema)
RUN dnf install -y python3.9 python3-pip && \
    alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 1 && \
    alternatives --set python3 /usr/bin/python3.9 && \
    python3 -m ensurepip --upgrade

# Agrega estas líneas ANTES de instalar SAM CLI
RUN pip3 install --upgrade pip setuptools wheel

# Instala dependencias de la capa en directorio temporal
RUN mkdir -p /tmp/layer-python && \
    pip3 install \
      --target /tmp/layer-python \
      "aws-lambda-powertools[Tracer,Logger,Metrics]" \
      simplejson \
      jsonpickle \
      aws_requests_auth

# Copia las dependencias al directorio de la capa
RUN mkdir -p /app/layers/python && \
    cp -r /tmp/layer-python/* /app/layers/python/

# 4. Instalar AWS CLI v2
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install && \
    rm -f awscliv2.zip && \
    rm -rf aws

# 5. Instalar SAM CLI
RUN wget https://github.com/aws/aws-sam-cli/releases/download/v1.64.0/aws-sam-cli-linux-x86_64.zip && \
    unzip aws-sam-cli-linux-x86_64.zip -d sam-installation && \
    ./sam-installation/install && \
    rm -rf aws-sam-cli-linux-x86_64.zip sam-installation

# 6. Instalar CDK y dependencias de Python (usando --user para evitar conflictos)
RUN npm install -g aws-cdk@2.40.0 && \
    python3 -m pip install --user --upgrade pip && \
    python3 -m pip install --user git-remote-codecommit==1.15.1 pylint==2.11.1 boto3

# Configurar PATH para incluir pip --user
ENV PATH="/root/.local/bin:${PATH}"

WORKDIR /app

# Verificar instalaciones
CMD ["bash", "-c", "aws --version && sam --version && cdk --version && node --version && python3 --version && jq --version"]

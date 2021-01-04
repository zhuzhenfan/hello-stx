FROM ubuntu:18.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
 && apt-get install -y wget gnupg2 vim

RUN wget https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS-2019.PUB \
 && apt-key add GPG-PUB-KEY-INTEL-SW-PRODUCTS-2019.PUB \
 && echo "deb https://apt.repos.intel.com/openvino/2019 all main" >> /etc/apt/sources.list.d/intel-openvino-2019.list

RUN apt update && apt-cache search openvino \
 && apt-cache search intel-openvino-runtime-ubuntu18 \
 && apt-get install intel-openvino-runtime-ubuntu18-2019.3.376  intel-openvino-dev-ubuntu18-2019.3.376 -y

RUN /opt/intel/openvino/install_dependencies/install_openvino_dependencies.sh \
 && apt-get install python3-setuptools python3-dev -y

RUN echo "source /opt/intel/openvino_2019.3.376/bin/setupvars.sh\nexport DISPLAY=:0" >> /root/.bashrc
RUN /opt/intel/openvino_2019.3.376/bin/setupvars.sh

RUN cp /opt/intel/openvino_2019.3.376/deployment_tools/model_optimizer/install_prerequisites/install_prerequisites.sh \
    /opt/intel/openvino_2019.3.376/deployment_tools/model_optimizer/install_prerequisites/nosudo_install_prerequisites.sh \
 && sed -i 's/sudo//g' /opt/intel/openvino_2019.3.376/deployment_tools/model_optimizer/install_prerequisites/nosudo_install_prerequisites.sh \
 && sed -i 's/-E //g' /opt/intel/openvino_2019.3.376/deployment_tools/model_optimizer/install_prerequisites/nosudo_install_prerequisites.sh

RUN /opt/intel/openvino_2019.3.376/deployment_tools/model_optimizer/install_prerequisites/nosudo_install_prerequisites.sh caffe \
 && /opt/intel/openvino_2019.3.376/deployment_tools/model_optimizer/install_prerequisites/nosudo_install_prerequisites.sh tf

RUN cp /opt/intel/openvino/deployment_tools/demo/demo_squeezenet_download_convert_run.sh \
    /opt/intel/openvino/deployment_tools/demo/nosudo_demo_squeezenet_download_convert_run.sh

COPY sque* /home/zzf/openvino_models/models/public/squeezenet1.1/

COPY sque* /root/openvino_models/models/public/squeezenet1.1/

RUN sed -i 's/sudo//g' /opt/intel/openvino/deployment_tools/demo/nosudo_demo_squeezenet_download_convert_run.sh \
 && sed -i 's/-E //g' /opt/intel/openvino/deployment_tools/demo/nosudo_demo_squeezenet_download_convert_run.sh \
 && sed -i 's/print_and_run "$python_binary" "$downloader_path" --name "$model_name" --output_dir "${models_path}" --cache_dir "${models_cache}"//g' \
    /opt/intel/openvino/deployment_tools/demo/nosudo_demo_squeezenet_download_convert_run.sh \
 && sed -i 's/install_prerequisites.sh/nosudo_install_prerequisites.sh/g' /opt/intel/openvino/deployment_tools/demo/nosudo_demo_squeezenet_download_convert_run.sh \
 && /opt/intel/openvino/deployment_tools/demo/nosudo_demo_squeezenet_download_convert_run.sh

RUN cp /opt/intel/openvino/deployment_tools/demo/demo_security_barrier_camera.sh \
    /opt/intel/openvino/deployment_tools/demo/nosudo_demo_security_barrier_camera.sh \
 && sed -i 's/sudo//g' /opt/intel/openvino/deployment_tools/demo/nosudo_demo_security_barrier_camera.sh \
 && sed -i 's/-E //g' /opt/intel/openvino/deployment_tools/demo/nosudo_demo_security_barrier_camera.sh \
 && sed -i "/print_and_run \.\/security_barrier_camera_demo/d" \
    /opt/intel/openvino/deployment_tools/demo/nosudo_demo_security_barrier_camera.sh

RUN /opt/intel/openvino/deployment_tools/demo/nosudo_demo_security_barrier_camera.sh

COPY nosudo_demo_security_barrier_camera.sh /opt/intel/openvino/deployment_tools/demo/start_demo_security_barrier_camera.sh

WORKDIR /opt/intel/openvino/deployment_tools/demo

CMD ["/bin/bash"]

FROM alpine:latest

RUN apk update && \
    apk add --no-cache nmap openssh && \
    echo "root:password" | chpasswd && \
    sed -i 's/#PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    mkdir /root/.ssh && \
    touch /root/.ssh/authorized_keys

EXPOSE 22
CMD ["sh", "-c", "rc-service sshd start && tail -f /dev/null"]

FROM alpine:3.14
LABEL "com.example.vendor"="deninet.com"

# Update the package list and install Ansible.
RUN apk -U upgrade &&\
    apk add --update --no-cache ansible git && \
    rm -rf /tmp/* \
           /var/cache/apk/* \
           /usr/lib/python3.6/site-packages/ansible/modules/network/ \
           /usr/lib/python3.6/site-packages/ansible/modules/cloud/ \
           /usr/lib/python3.6/site-packages/ansible/modules/windows/

# Copy the Ansible configuration files
COPY ansible-hosts /etc/ansible/hosts
COPY ansible.cfg /etc/ansible/ansible.cfg
COPY ansible /ansible

# Run the build.
RUN ansible-galaxy install -fr /ansible/requirements.yml && \
    ansible-playbook -i /ansible/inventories/all.ini /ansible/build.yml && \
    apk del git

# Configure the runtime environment of the container.
EXPOSE 6081 6082

# Switch to the service account.
USER varnish

# Set the entrypoint and default command.
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["docker-cmd.sh", "/var/lib/varnish/varnish-foreground.sh"]

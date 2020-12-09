ARG REGISTRY
ARG IMAGE
FROM $REGISTRY/$IMAGE
COPY root-fs.sh /

COPY --chown=root:root chroot-script /etc/debootstrap/
COPY --chown=root:root config /etc/debootstrap/
COPY --chown=root:root packages /etc/debootstrap/

ENTRYPOINT ["./root-fs.sh"]

ARG BUILD_ARCH=amd64
FROM debian:bookworm-slim

ARG BUILD_ARCH
ARG TTYD_VERSION=1.7.7

ENV LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_INSTALL_DIR=/opt/uv-python \
    UV_TOOL_DIR=/opt/uv-tools \
    UV_TOOL_BIN_DIR=/usr/local/bin \
    VIBE_HOME=/data/vibe \
    PATH=/usr/local/bin:/usr/bin:/bin:/sbin

SHELL ["/bin/bash", "-e", "-o", "pipefail", "-c"]

# ---- system packages -------------------------------------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
        bash ca-certificates coreutils curl git jq nano \
        openssh-client python3 python3-pip ripgrep tmux tzdata xz-utils \
    && rm -rf /var/lib/apt/lists/*

# ---- ttyd: static binary (not in Debian repos) ------------------------------
RUN case "${BUILD_ARCH}" in \
        aarch64) TTYD_ASSET="ttyd.aarch64" ;; \
        amd64)   TTYD_ASSET="ttyd.x86_64" ;; \
        i386)    TTYD_ASSET="ttyd.i686" ;; \
        *)       echo "unsupported BUILD_ARCH: ${BUILD_ARCH}" >&2; exit 1 ;; \
    esac && \
    curl -fsSL \
      "https://github.com/tsl0922/ttyd/releases/download/${TTYD_VERSION}/${TTYD_ASSET}" \
      -o /usr/local/bin/ttyd \
 && chmod +x /usr/local/bin/ttyd

# ---- s6-overlay v2 (native /etc/cont-init.d + /etc/services.d support) -----
# MUST be extracted to / — it provides /init, the container entrypoint.
RUN case "${BUILD_ARCH}" in \
        aarch64) S6_ARCH="aarch64" ;; \
        *)       S6_ARCH="amd64" ;; \
    esac && \
    curl -fsSL \
      "https://github.com/just-containers/s6-overlay/releases/download/v2.1.0.2/s6-overlay-${S6_ARCH}.tar.gz" \
      -o /tmp/s6.tar.gz \
 && tar -xzf /tmp/s6.tar.gz -C / \
 && rm /tmp/s6.tar.gz

# ---- uv + managed Python + CLIs ---------------------------------------------
RUN curl -LsSf https://astral.sh/uv/install.sh \
      | env UV_INSTALL_DIR=/usr/local/bin sh \
 && uv python install 3.13 \
 && uv tool install --python 3.13 mistral-vibe \
 && python3 -m pip install --break-system-packages --no-cache-dir fastmcp httpx \
 && rm -rf /root/.cache /tmp/*

# ---- Home Assistant CLI -----------------------------------------------------
RUN curl -fsSL "https://github.com/home-assistant/cli/releases/latest/download/ha_${BUILD_ARCH}" \
        -o /usr/local/bin/ha && chmod +x /usr/local/bin/ha

# ---- app rootfs -------------------------------------------------------------
COPY rootfs/ /

RUN chmod a+x \
        /usr/bin/vibe-launcher \
        /etc/cont-init.d/10-init-vibe.sh \
        /etc/services.d/vibe-terminal/run \
        /etc/services.d/vibe-terminal/finish

ENTRYPOINT ["/init"]

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -sf http://localhost:7681/ || exit 1

LABEL \
    io.hass.name="Mistral Vibe" \
    io.hass.description="Run Mistral Vibe CLI directly inside Home Assistant" \
    io.hass.type="app" \
    maintainer="Paul de Vries <BillyBlaze@users.noreply.github.com>"
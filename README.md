docker buildx build --platform linux/amd64,linux/arm64/v8 --no-cache -t ghcr.io/rowa78/eve-online-tool-suite/eve-sde-mariadb:20220524 --push .

docker run -d --restart unless-stopped -p 3307:3306 ghcr.io/rowa78/eve-online-tool-suite/eve-sde-mariadb:20220524


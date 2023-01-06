# WPScan WordPress Security Scanner

## What is WPScan?

The WPScan CLI tool is a free, for non-commercial use, black box WordPress security scanner written for security professionals and blog maintainers to test the security of their sites.

https://wpscan.com/wordpress-security-scanner

## Run WPScan in a Docker container

```
docker run -it --rm \
--mount type=bind,source=$HOME/temp/docker-bind,target=/output \
wpscanteam/wpscan -o /output/wpscan-output.txt \
--url <url of wp site to scan> --enumerate u \
--api-token <api token>
```
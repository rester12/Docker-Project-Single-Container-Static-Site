\# Docker Project 1: Single-Container Static Site with Nginx



\## Overview



This hands-on portfolio project demonstrates how to deploy, inspect, customize, validate, and troubleshoot containerized web servers using Docker Desktop on Windows.



The project follows a business scenario in which \*\*BrightPath Realty\*\* needs a marketing website deployed quickly without provisioning, patching, or maintaining a traditional virtual machine. The implementation begins with the official Nginx container image, validates the default web service, replaces the default content with a custom static website, and then progressively improves the deployment by introducing bind-mounted content and externalized Nginx configuration.



The project also runs Apache HTTP Server alongside Nginx to compare container configuration paths, document roots, and HTTP behavior. The final Nginx deployment uses read-only bind mounts, gzip compression, and a custom HTTP 404 error page.



This is a project-based learning environment designed to demonstrate Docker fundamentals, Linux container inspection, HTTP validation, web-server configuration, troubleshooting, storage concepts, and technical documentation. It is not presented as a production deployment.



\## Medium Article



A detailed Medium walkthrough documenting the complete build process, screenshots, troubleshooting, validation steps, engineering decisions, and lessons learned is available here:

Building a Containerized Static Website with Docker, Nginx, Bind Mounts, Apache, Gzip, and Custom Error Handling


\## Architecture



```mermaid

graph LR

&#x20;   A\[Windows Host] --> B\[Docker Desktop]

&#x20;   B --> C\[Nginx Container]

&#x20;   B --> D\[Apache Container]



&#x20;   E\[BrightPath Site Files] -->|Read-Only Bind Mount| C

&#x20;   F\[Custom nginx.conf] -->|Read-Only Bind Mount| C



&#x20;   G\[Browser / curl] -->|Host Port 8080| C

&#x20;   G -->|Host Port 8081| D



&#x20;   C -->|Container Port 80| H\[BrightPath Realty Site]

&#x20;   D -->|Container Port 80| I\[Apache Default Site]

```



\## Technologies Used



\* Docker Desktop

\* Docker Engine and Docker CLI

\* Windows PowerShell

\* Nginx

\* Apache HTTP Server (`httpd`)

\* Linux containers

\* HTML5

\* CSS3

\* HTTP and curl

\* Docker bind mounts

\* Custom Nginx configuration

\* Git and GitHub

\* Markdown and Mermaid diagrams



\## Project Objectives



\* Pull and run an official Nginx container image.

\* Run the Nginx container in detached mode.

\* Publish container port `80` through Windows host port `8080`.

\* Validate the Nginx service through a web browser and `curl`.

\* Inspect container logs, filesystem paths, configuration, and running processes.

\* Replace the default Nginx landing page with a custom BrightPath Realty website.

\* Compare container-local content with host-mounted content.

\* Rebuild the Nginx deployment using a read-only Docker bind mount.

\* Demonstrate live content changes without rebuilding the image or recreating the container.

\* Run Nginx and Apache side by side using separate host ports.

\* Compare Nginx and Apache document roots and configuration structures.

\* Externalize the Nginx configuration through a second read-only bind mount.

\* Enable gzip compression.

\* Configure a custom HTTP 404 error page.

\* Validate configuration changes with `nginx -t`.

\* Verify HTTP `200`, `404`, and gzip behavior with `curl`.

\* Practice troubleshooting without unnecessarily modifying a minimal container image.



\## Repository Contents



```text

.

|-- .gitignore

|-- README.md

|-- nginx/

|   `-- nginx.conf

`-- site/

&#x20;   |-- 404.html

&#x20;   |-- index.html

&#x20;   `-- styles.css

```



\## Business Scenario



BrightPath Realty needs a lightweight marketing website available quickly without provisioning and patching a dedicated virtual machine.



A containerized architecture was selected to demonstrate how an existing web-server image can be deployed rapidly, replaced just as quickly, and configured through external host files.



The project progressively evolves from a basic container deployment into a more maintainable architecture where website content and server configuration remain outside the container lifecycle.



\## Phase 1: Docker Environment and Nginx Image



Docker Desktop was used as the local container runtime on Windows.



The Docker client and engine were validated with:



```powershell

docker --version

docker version

docker info

docker images

docker ps

docker ps -a

```



The official Nginx image was then downloaded:



```powershell

docker pull nginx:latest

```



The locally available image was verified with:



```powershell

docker images nginx

```



At this point, the Nginx image existed locally but no BrightPath container had been created.



An \*\*image\*\* is the packaged template used to create containers, while a \*\*container\*\* is a running or stopped instance of an image.



\## Phase 2: Initial Nginx Container Deployment



The first Nginx container was created in detached mode:



```powershell

docker run -d --name brightpath-nginx -p 8080:80 nginx:latest

```



The port mapping:



```text

8080:80

```



publishes Windows host port `8080` and forwards traffic to port `80` inside the Nginx container.



The running container was verified with:



```powershell

docker ps

```



Container state and image metadata were also inspected:



```powershell

docker inspect --format='{{.State.Status}}' brightpath-nginx

docker inspect --format='{{.Config.Image}}' brightpath-nginx

docker inspect --format='{{json .NetworkSettings.Ports}}' brightpath-nginx

```



The default Nginx site was available at:



```text

http://localhost:8080

```



\## Phase 3: HTTP Validation and Logging



The web service was validated independently of the browser using curl:



```powershell

curl.exe http://localhost:8080

curl.exe -I http://localhost:8080

```



The HTTP headers confirmed a successful response:



```text

HTTP/1.1 200 OK

Server: nginx

```



Container logs were inspected with:



```powershell

docker logs brightpath-nginx

```



Live log monitoring was also practiced:



```powershell

docker logs -f brightpath-nginx

```



Refreshing the browser generated new Nginx access-log entries, connecting browser requests with server-side observability.



\## Phase 4: Inspecting the Running Container



An interactive shell was opened inside the running container:



```powershell

docker exec -it brightpath-nginx /bin/bash

```



The container operating system and filesystem were inspected:



```bash

cat /etc/os-release

pwd

ls -la

```



Nginx configuration files were located under:



```text

/etc/nginx

```



The default configuration was inspected with:



```bash

cat /etc/nginx/nginx.conf

cat /etc/nginx/conf.d/default.conf

```



The default server configuration showed that Nginx listened on container port `80` and served static content from:



```text

/usr/share/nginx/html

```



The default document root was inspected with:



```bash

ls -la /usr/share/nginx/html

cat /usr/share/nginx/html/index.html

```



This connected the browser-visible Nginx welcome page to the actual file being served inside the container.



\## Troubleshooting: Minimal Container and Missing `ps`



While inspecting processes inside the Nginx container, the following command was attempted:



```bash

ps aux

```



The container returned:



```text

bash: ps: command not found

```



The official Nginx image is intentionally minimal and did not include the traditional `ps` process-inspection utility.



Rather than installing additional packages into the running workload solely for troubleshooting, the container shell was exited and Docker-native process inspection was used:



```powershell

docker top brightpath-nginx

```



This displayed the Nginx master and worker processes without modifying the container.



Nginx itself was also validated directly:



```powershell

docker exec brightpath-nginx nginx -v

docker exec brightpath-nginx nginx -t

```



This troubleshooting step reinforced an important container principle: a missing administrative utility does not mean the workload has failed, and container-native tooling can often provide the required observability without increasing the image or runtime footprint.



\## Phase 5: Custom BrightPath Realty Website



The custom website was created on the Windows host under:



```text

site/

```



The website consists of:



```text

index.html

styles.css

```



The first deployment method manually copied those files into the existing container:



```powershell

docker cp .\\site\\index.html brightpath-nginx:/usr/share/nginx/html/index.html

docker cp .\\site\\styles.css brightpath-nginx:/usr/share/nginx/html/styles.css

```



The files were verified inside the container:



```powershell

docker exec brightpath-nginx ls -la /usr/share/nginx/html

```



The custom site was then validated through the browser and curl:



```powershell

curl.exe http://localhost:8080

curl.exe http://localhost:8080 | Select-String "BrightPath Realty"

curl.exe -I http://localhost:8080

```



The Nginx access logs also showed requests for both the HTML page and stylesheet.



\## Phase 6: Moving from Container-Local Files to a Bind Mount



Although `docker cp` successfully replaced the default website, the copied files became part of the individual container's writable filesystem.



That means deleting the container would also delete those container-local changes.



Before redesigning the deployment, the mount state was inspected:



```powershell

docker inspect --format='{{json .Mounts}}' brightpath-nginx

```



The original container was then stopped and removed:



```powershell

docker stop brightpath-nginx

docker rm brightpath-nginx

```



The BrightPath source files remained safely on the Windows host.



A new Nginx container was created using a read-only bind mount:



```powershell

docker run -d `

&#x20; --name brightpath-nginx `

&#x20; -p 8080:80 `

&#x20; --mount type=bind,source="${PWD}\\site",target=/usr/share/nginx/html,readonly `

&#x20; nginx:latest

```



The mount connected:



```text

Windows:

C:\\Docker-Projects\\brightpath-static-site\\site

```



to:



```text

Container:

/usr/share/nginx/html

```



The mount was verified with:



```powershell

docker inspect --format='{{json .Mounts}}' brightpath-nginx

```



The container now served the BrightPath site directly from the Windows project directory without using `docker cp`.



\## Bind Mount vs. Named Volume



A \*\*bind mount\*\* maps a specific path from the host filesystem into the container.



In this project:



```text

C:\\Docker-Projects\\brightpath-static-site\\site

```



was mapped to:



```text

/usr/share/nginx/html

```



This was appropriate because the website source code is intentionally managed from the Windows project directory.



A \*\*named volume\*\* is storage managed by Docker rather than a specific user-selected host directory.



Named volumes are often useful when persistent application data should survive container replacement without requiring direct interaction with a particular host filesystem path.



The bind mount was selected here because direct host-side editing of static website files was part of the required workflow.



\## Phase 7: Live Bind-Mount Content Updates



To demonstrate the bind mount behavior, the BrightPath headline was modified directly in the Windows host file.



No image rebuild, `docker cp`, container restart, or container recreation was performed.



After saving the host file, refreshing:



```text

http://localhost:8080

```



immediately displayed the updated content.



The change was also validated with curl:



```powershell

curl.exe http://localhost:8080 | Select-String "trusted path"

```



This demonstrated that Nginx was reading the site files directly through the bind mount.



\## Phase 8: Nginx and Apache Side by Side



The official Apache HTTP Server image was downloaded:



```powershell

docker pull httpd:latest

```



Apache was started on host port `8081`:



```powershell

docker run -d `

&#x20; --name brightpath-apache `

&#x20; -p 8081:80 `

&#x20; httpd:latest

```



Nginx and Apache could run simultaneously because each container had its own network namespace and both could listen internally on port `80`.



The Windows host ports were different:



```text

Nginx  -> localhost:8080 -> container port 80

Apache -> localhost:8081 -> container port 80

```



Both containers were verified with:



```powershell

docker ps

```



Their HTTP headers were compared:



```powershell

curl.exe -I http://localhost:8080

curl.exe -I http://localhost:8081

```



The responses identified the corresponding server implementations.



\## Nginx vs. Apache Directory Structure



The Nginx document root used in this project was:



```text

/usr/share/nginx/html

```



The Apache HTTPD image served default content from:



```text

/usr/local/apache2/htdocs

```



Apache's primary configuration was located under:



```text

/usr/local/apache2/conf/httpd.conf

```



while Nginx used:



```text

/etc/nginx/nginx.conf

/etc/nginx/conf.d/

```



The directory structures were compared with:



```powershell

docker exec brightpath-nginx ls -la /etc/nginx

docker exec brightpath-apache ls -la /usr/local/apache2/conf



docker exec brightpath-nginx ls -la /usr/share/nginx/html

docker exec brightpath-apache ls -la /usr/local/apache2/htdocs

```



This exercise demonstrated that different containerized web servers can expose similar services while using different filesystem and configuration conventions internally.



\## Phase 9: Custom Nginx Configuration



A custom configuration file was created at:



```text

nginx/nginx.conf

```



The configuration enabled gzip compression, defined the BrightPath document root, configured a custom 404 page, and used `try\_files` to return an HTTP 404 when a requested resource did not exist.



Important directives included:



```nginx

gzip on;

```



and:



```nginx

error\_page 404 /404.html;

```



with:



```nginx

location / {

&#x20;   try\_files $uri $uri/ =404;

}

```



A custom error page was added:



```text

site/404.html

```



\## Phase 10: Externalizing Both Content and Configuration



The Nginx container was recreated using two read-only bind mounts:



```powershell

docker run -d `

&#x20; --name brightpath-nginx `

&#x20; -p 8080:80 `

&#x20; --mount type=bind,source="${PWD}\\site",target=/usr/share/nginx/html,readonly `

&#x20; --mount type=bind,source="${PWD}\\nginx\\nginx.conf",target=/etc/nginx/nginx.conf,readonly `

&#x20; nginx:latest

```



The resulting architecture externalized both:



\* application content;

\* server configuration.



The mount configuration was verified with:



```powershell

docker inspect --format='{{json .Mounts}}' brightpath-nginx

```



The Nginx configuration was validated before relying on the service:



```powershell

docker exec brightpath-nginx nginx -t

```



A successful result confirmed that the configuration syntax was valid.



\## Phase 11: Custom 404 Validation



A nonexistent resource was requested:



```text

http://localhost:8080/property-does-not-exist

```



The browser displayed the custom BrightPath Realty error page.



The HTTP status was independently validated:



```powershell

curl.exe -I http://localhost:8080/property-does-not-exist

```



The server correctly returned:



```text

HTTP/1.1 404 Not Found

```



This confirmed that the custom user experience did not incorrectly return an HTTP `200 OK`.



The request was also visible in Nginx logs:



```powershell

docker logs --tail 20 brightpath-nginx

```



\## Phase 12: Gzip Compression Validation



The custom Nginx configuration enabled gzip for selected text-based content types.



Compression was tested by explicitly telling Nginx that the client supported gzip:



```powershell

curl.exe -I -H "Accept-Encoding: gzip" http://localhost:8080/styles.css

```



The response included:



```text

Content-Encoding: gzip

```



This demonstrated successful HTTP content negotiation and server-side compression.



A standard request was also compared against the gzip-enabled request:



```powershell

curl.exe -I http://localhost:8080/styles.css

curl.exe -I -H "Accept-Encoding: gzip" http://localhost:8080/styles.css

```



\## Validation



The completed Nginx deployment was validated with:



```powershell

docker ps

docker exec brightpath-nginx nginx -t

curl.exe -I http://localhost:8080

curl.exe -I -H "Accept-Encoding: gzip" http://localhost:8080/styles.css

curl.exe -I http://localhost:8080/property-does-not-exist

```



Validation confirmed:



\* The Nginx container was running on host port `8080`.

\* The Apache container was running independently on host port `8081`.

\* Nginx configuration syntax was valid.

\* The BrightPath homepage returned HTTP `200 OK`.

\* Static CSS content could be returned with gzip compression.

\* Invalid resources returned HTTP `404 Not Found`.

\* The custom BrightPath 404 page was displayed for missing resources.

\* Website source files remained on the Windows host.

\* Nginx consumed website content through a read-only bind mount.

\* The custom `nginx.conf` was also provided through a read-only bind mount.

\* Host-side site changes could be served without rebuilding the image.



\## Engineering Decisions



\### Official Images



Official Nginx and Apache images were used rather than building custom images for the initial learning objectives.



\### Detached Containers



The web servers were run in detached mode so they continued operating independently of the active PowerShell session.



\### Explicit Port Publishing



Host ports `8080` and `8081` were used to make the containerized services independently reachable from Windows.



\### Read-Only Bind Mounts



The content and Nginx configuration mounts were configured as read-only because the web-server container only needed to consume those files.



\### External Configuration



The final `nginx.conf` remained in the project directory instead of being edited manually inside the running container. This improves reproducibility and makes configuration suitable for version control.



\### Docker-Native Troubleshooting



When the minimal Nginx image lacked `ps`, `docker top` was used rather than adding unnecessary packages to the running workload.



\## Security and Operational Considerations



\* Use official or otherwise trusted container images.

\* Pin specific image versions or immutable digests for production deployments instead of relying indefinitely on `latest`.

\* Use read-only mounts when containers do not need write access.

\* Do not store secrets, passwords, API keys, or tokens inside HTML files or Git repositories.

\* Keep configuration under version control.

\* Validate Nginx configuration with `nginx -t` before reload or deployment.

\* Inspect container logs when troubleshooting HTTP requests.

\* Limit published host ports to services that must be externally reachable.

\* Treat containers as replaceable workloads rather than manually maintained servers.

\* Avoid installing unnecessary troubleshooting packages into minimal production-oriented images.



\## Lessons Learned



\* Docker images and containers represent different lifecycle states.

\* Port publishing connects a host port to a container service port.

\* A successful browser request should also be validated at the HTTP level.

\* `docker logs` provides direct visibility into containerized application activity.

\* `docker exec` allows targeted inspection and command execution inside running containers.

\* Minimal container images may intentionally omit traditional administration tools.

\* Docker-native commands such as `docker top` can provide observability without modifying the workload.

\* Manually copying application files into a container works but ties those changes to that container's lifecycle.

\* Bind mounts make host-managed source files available directly inside a container.

\* Read-only mounts reduce unnecessary write capability.

\* Nginx and Apache provide similar HTTP services while using different default configuration and document-root structures.

\* Externalizing server configuration improves reproducibility.

\* `nginx -t` provides a critical configuration validation step.

\* Custom error pages should preserve the correct HTTP status code.

\* HTTP gzip behavior should be proven from response headers rather than assumed from configuration alone.



\## How to Run the Project



\### Prerequisites



\* Windows with Docker Desktop

\* Docker Engine running

\* PowerShell

\* Git, if cloning from GitHub



Clone the repository and enter it:



```powershell

git clone <repository-url>

cd Docker-Project-1-Single-Container-Static-Site

```



Start Nginx:



```powershell

docker run -d `

&#x20; --name brightpath-nginx `

&#x20; -p 8080:80 `

&#x20; --mount type=bind,source="${PWD}\\site",target=/usr/share/nginx/html,readonly `

&#x20; --mount type=bind,source="${PWD}\\nginx\\nginx.conf",target=/etc/nginx/nginx.conf,readonly `

&#x20; nginx:latest

```



Optional Apache comparison:



```powershell

docker run -d `

&#x20; --name brightpath-apache `

&#x20; -p 8081:80 `

&#x20; httpd:latest

```



Open:



```text

http://localhost:8080

```



Optional Apache endpoint:



```text

http://localhost:8081

```



Validate:



```powershell

docker ps

docker exec brightpath-nginx nginx -t

curl.exe -I http://localhost:8080

curl.exe -I -H "Accept-Encoding: gzip" http://localhost:8080/styles.css

curl.exe -I http://localhost:8080/property-does-not-exist

```



Stop and remove the containers when finished:



```powershell

docker stop brightpath-nginx brightpath-apache

docker rm brightpath-nginx brightpath-apache

```



\## Future Improvements



\* Create a custom Dockerfile that packages the website into an immutable application image.

\* Add a `.dockerignore`.

\* Pin specific Nginx and Apache image versions instead of using `latest`.

\* Add Docker Compose to manage the Nginx and Apache services declaratively.

\* Add a health check for the Nginx container.

\* Add automated HTML or configuration validation through GitHub Actions.

\* Scan container images for known vulnerabilities.

\* Add structured log collection or monitoring.

\* Add TLS termination for HTTPS.

\* Deploy the containerized site to a cloud container service.

\* Compare bind-mounted development workflows with immutable-image deployment patterns.




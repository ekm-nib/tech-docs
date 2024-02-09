## How to podman containes in Windows 11

1. Install Podman CLI for Windows
```
https://github.com/containers/podman/releases/download/v4.9.2/podman-4.9.2-setup.exe
```
2. Install Python
```
https://www.python.org/ftp/python/3.12.2/python-3.12.2-amd64.exe
```
3. Go Project Directory
```
cd .\project-directory
```
4. Initialise python virtual environment
```
python -m venv venv
venv\Scripts\ativate.bat
```
5. Install podman-compose using pip
```
pip install podman-compose
```
5. Initialise a new Virtual machine
```
podman machine init
```
6. Start the Virtula machine
```
podman machine start
```
7. Create Dockerfile & docker-compose.yml file

8. Run container using podman-compose
```
podman-compose up -d
```


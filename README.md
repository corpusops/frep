
DISCLAIMER - ABANDONED/UNMAINTAINED CODE / DO NOT USE
=======================================================
While this repository has been inactive for some time, this formal notice, issued on **December 10, 2024**, serves as the official declaration to clarify the situation. Consequently, this repository and all associated resources (including related projects, code, documentation, and distributed packages such as Docker images, PyPI packages, etc.) are now explicitly declared **unmaintained** and **abandoned**.

I would like to remind everyone that this project’s free license has always been based on the principle that the software is provided "AS-IS", without any warranty or expectation of liability or maintenance from the maintainer.
As such, it is used solely at the user's own risk, with no warranty or liability from the maintainer, including but not limited to any damages arising from its use.

Due to the enactment of the Cyber Resilience Act (EU Regulation 2024/2847), which significantly alters the regulatory framework, including penalties of up to €15M, combined with its demands for **unpaid** and **indefinite** liability, it has become untenable for me to continue maintaining all my Open Source Projects as a natural person.
The new regulations impose personal liability risks and create an unacceptable burden, regardless of my personal situation now or in the future, particularly when the work is done voluntarily and without compensation.

**No further technical support, updates (including security patches), or maintenance, of any kind, will be provided.**

These resources may remain online, but solely for public archiving, documentation, and educational purposes.

Users are strongly advised not to use these resources in any active or production-related projects, and to seek alternative solutions that comply with the new legal requirements (EU CRA).

**Using these resources outside of these contexts is strictly prohibited and is done at your own risk.**

Regarding the potential transfer of the project to another entity, discussions are ongoing, but no final decision has been made yet. As a last resort, if the project and its associated resources are not transferred, I may begin removing any published resources related to this project (e.g., from PyPI, Docker Hub, GitHub, etc.) starting **March 15, 2025**, especially if the CRA’s risks remain disproportionate.


[![Build Status](https://travis-ci.org/subchen/frep.svg?branch=master)](https://travis-ci.org/subchen/frep)
[![License](http://img.shields.io/badge/License-Apache_2-red.svg?style=flat)](http://www.apache.org/licenses/LICENSE-2.0)


# frep

Generate file using template from environment, arguments, json/yaml/toml config files.

```
NAME:
   frep - Generate file using template

USAGE:
   frep [options] input-file[:output-file] ...

VERSION:
   1.3.4-x

AUTHORS:
   Guoqiang Chen <subchen@gmail.com>

OPTIONS:
   -e, --env name=value    set variable name=value, can be passed multiple times
       --json jsonstring   load variables from json object string
       --load file         load variables from json/yaml/toml file
       --overwrite         overwrite if destination file exists
       --dryrun            just output result to console instead of file
       --delims value      template tag delimiters (default: {{:}})
       --help              print this usage
       --version           print version information

EXAMPLES:
   frep nginx.conf.in -e webroot=/usr/share/nginx/html -e port=8080
   frep nginx.conf.in:/etc/nginx.conf -e webroot=/usr/share/nginx/html -e port=8080
   frep nginx.conf.in --json '{"webroot": "/usr/share/nginx/html", "port": 8080}'
   frep nginx.conf.in --load config.json --overwrite
   echo "{{ .Env.PATH }}"  | frep -
```

## Downloads

v1.3.4 Release: https://github.com/subchen/frep/releases/tag/v1.3.4

- Linux

    ```
    curl -fSL https://github.com/subchen/frep/releases/download/v1.3.4/frep-1.3.4-linux-amd64 -o /usr/local/bin/frep
    chmod +x /usr/local/bin/frep
    ```

- macOS

    ```
    brew install subchen/tap/frep
    ```

- Windows

    ```
    wget https://github.com/subchen/frep/releases/download/v1.3.4/frep-1.3.4-windows-amd64.exe
    ```

## Docker

You can run frep using docker container

```
docker run -it --rm subchen/frep:1.3.4 --help
```


## Examples

### Load template variables

- Load from environment

    ```
    export webroot=/usr/share/nginx/html
    export port=8080
    frep nginx.conf.in
    ```

- Load from arguments

    ```
    frep nginx.conf.in -e webroot=/usr/share/nginx/html -e port=8080
    ```

- Load from JSON String

    ```
    frep nginx.conf.in --json '{"webroot": "/usr/share/nginx/html", "port": 8080}'
    ```

- Load from JSON file

    ```
    cat > config.json << EOF
    {
      "webroot": "/usr/share/nginx/html",
      "port": 8080,
      "servers": [
        "127.0.0.1:8081",
        "127.0.0.1:8082"
      ]
    }
    EOF

    frep nginx.conf.in --load config.json
    ```

- Load from YAML file

    ```
    cat > config.yaml << EOF
    webroot: /usr/share/nginx/html
    port: 8080
    servers:
      - 127.0.0.1:8081
      - 127.0.0.1:8082
    EOF

    frep nginx.conf.in --load config.yaml
    ```

- Load from TOML file

    ```
    cat > config.toml << EOF
    webroot = /usr/share/nginx/html
    port = 8080
    servers = [
       "127.0.0.1:8081",
       "127.0.0.1:8082"
    ]
    EOF

    frep nginx.conf.in --load config.toml
    ```

### Input/Output

- Input from file

    ```
    // input file: nginx.conf
    frep nginx.conf.in
    ```

- Input from console(stdin)

    ```
    // input from stdin pipe
    echo "{{ .Env.PATH }}" | frep -
    ```

- Output to default file (Removed last file ext)

    ```
    // output file: nginx.conf
    frep nginx.conf.in --overwrite
    ```

- Output to the specified file

    ```
    // output file: /etc/nginx.conf
    frep nginx.conf.in:/etc/nginx.conf --overwrite -e port=8080
    ```

- Output to console(stdout)

    ```
    frep nginx.conf.in --dryrun
    frep nginx.conf.in:-
    ```

- Output multiple files

    ```
    frep nginx.conf.in redis.conf.in ...
    ```

## Template

Templates use Golang [text/template](http://golang.org/pkg/text/template/).

You can access environment variables within a template

```
Env.PATH = {{ .Env.PATH }}
```

If your template file uses `{{` and `}}` as part of it's syntax,
you can change the template escape characters using the `--delims`.

```
frep --delims "<%:%>" ...
```

There are some built-in functions as well: Masterminds/sprig v2.14.1
- github: https://github.com/Masterminds/sprig
- doc: http://masterminds.github.io/sprig/

More funcs added:
- toJson
- toYaml
- toToml
- toBool
- fileSize
- fileLastModified
- fileGetBytes
- fileGetString

Sample of nginx.conf.in

```
server {
    listen {{.port}} default_server;

    root {{.webroot | default "/usr/share/nginx/html"}};
    index index.html index.htm;

    location /api {
        access_log off;
        proxy_pass http://backend;
    }
}

upstream backend {
    ip_hash;
{{- range .servers }}
    server {{.}};
{{- end }}
}
```

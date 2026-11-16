## TP Sécurisation d'une image et d'un conteneur Docker

#### Quel utilisateur éxécute le conteneur :

``` bash
whoami
root
```

#### Image de base utilisée : 

``` Dockerfile
ARG NODE_VERSION=22.23.5-alpine3.24@sha256:1314fb30ec37592d5845e5a160d522c0e680972aa534549940b8d73bde88481a
FROM node:${NODE_VERSION}
```

#### Taille de l'image :

Totale :

``` bash
images quiz-docker:initial --format "{{.Size}}"
320MB
```

#### Par couche : 

``` bash
docker history quiz-docker:initial
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
e787944646e4   2 days ago    CMD ["serve" "-s" "dist"]                       0B        buildkit.dockerfile.v0
<missing>      2 days ago    EXPOSE [3000/tcp]                               0B        buildkit.dockerfile.v0
<missing>      2 days ago    RUN /bin/sh -c npm run build && rm -rf node_…   81.9kB    buildkit.dockerfile.v0
<missing>      2 days ago    COPY . . # buildkit                             16.9MB    buildkit.dockerfile.v0
<missing>      3 days ago    RUN /bin/sh -c npm ci # buildkit                28.8MB    buildkit.dockerfile.v0
<missing>      3 days ago    COPY package*.json ./ # buildkit                28.7kB    buildkit.dockerfile.v0
<missing>      3 days ago    RUN /bin/sh -c npm install -g serve # buildk…   18.9MB    buildkit.dockerfile.v0
<missing>      3 days ago    WORKDIR /app                                    8.19kB    buildkit.dockerfile.v0
<missing>      4 days ago    CMD ["node"]                                    0B        buildkit.dockerfile.v0
<missing>      4 days ago    ENTRYPOINT ["docker-entrypoint.sh"]             0B        buildkit.dockerfile.v0
<missing>      4 days ago    COPY docker-entrypoint.sh /usr/local/bin/ # …   20.5kB    buildkit.dockerfile.v0
<missing>      4 days ago    RUN /bin/sh -c apk add --no-cache --virtual …   5.49MB    buildkit.dockerfile.v0
<missing>      4 days ago    ENV YARN_VERSION=1.22.22                        0B        buildkit.dockerfile.v0
<missing>      4 days ago    RUN /bin/sh -c addgroup -g 1000 node     && …   158MB     buildkit.dockerfile.v0
<missing>      4 days ago    ENV NODE_VERSION=22.23.5                        0B        buildkit.dockerfile.v0
<missing>      10 days ago   CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      10 days ago   ADD alpine-minirootfs-3.24.4-aarch64.tar.gz …   9.31MB    buildkit.dockerfile.v0
```

#### Analyse de sécurité :

- Contneur exec avec root
- Image qui embarque des binaires inutiles
- Pas de .dockerignore
- Des vulnérabilités node.js sont présentes

#### Hadolint :

```` bash
hadolint Dockerfile
Dockerfile:7 DL3016 warning: Pin versions in npm. Instead of `npm install <package>` use `npm install <package>@<version>`
````

#### Trivy :

``` bash
 trivy image -q --severity HIGH,CRITICAL quiz-docker:initial

Report Summary

┌──────────────────────────────────────────────────────────────────────────────────┬──────────┬─────────────────┬─────────┐
│                                      Target                                      │   Type   │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ quiz-docker:initial (alpine 3.24.4)                                              │  alpine  │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ app/package.json                                                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ opt/yarn-v1.22.22/package.json                                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/corepack/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/node_modules/ansi-reg- │ node-pkg │        0        │    -    │
│ ex/package.json                                                                  │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/node_modules/emoji-re- │ node-pkg │        0        │    -    │
│ gex/package.json                                                                 │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/node_modules/string-w- │ node-pkg │        0        │    -    │
│ idth/package.json                                                                │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/node_modules/strip-an- │ node-pkg │        0        │    -    │
│ si/package.json                                                                  │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/fs-minipass/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/string-locale-compare/packa- │ node-pkg │        0        │    -    │
│ ge.json                                                                          │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/agent/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/arborist/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/config/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/fs/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/git/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/installed-package-contents/- │ node-pkg │        0        │    -    │
│ package.json                                                                     │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/map-workspaces/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/metavuln-calculator/node_mo- │ node-pkg │        1        │    -    │
│ dules/pacote/package.json                                                        │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/metavuln-calculator/package- │ node-pkg │        0        │    -    │
│ .json                                                                            │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/name-from-folder/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/node-gyp/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/package-json/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/promise-spawn/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/query/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/redact/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/run-script/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@pkgjs/parseargs/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@sigstore/bundle/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@sigstore/core/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@sigstore/protobuf-specs/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@sigstore/sign/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@sigstore/tuf/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@sigstore/verify/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@tufjs/canonical-json/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/abbrev/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/agent-base/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ansi-regex/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ansi-styles/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/aproba/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/archy/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/balanced-match/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/bin-links/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/binary-extensions/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/brace-expansion/package.json         │ node-pkg │        5        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cacache/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/chalk/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/chownr/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ci-info/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cidr-regex/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cli-columns/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cmd-shim/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/color-convert/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/color-name/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/common-ancestor-path/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cross-spawn/node_modules/which/pack- │ node-pkg │        0        │    -    │
│ age.json                                                                         │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cross-spawn/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cssesc/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/debug/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/diff/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/eastasianwidth/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/emoji-regex/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/encoding/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/env-paths/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/err-code/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/exponential-backoff/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/fastest-levenshtein/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/fdir/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/foreground-child/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/fs-minipass/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/glob/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/graceful-fs/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/hosted-git-info/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/http-cache-semantics/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/http-proxy-agent/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/https-proxy-agent/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/iconv-lite/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ignore-walk/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/imurmurhash/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ini/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/init-package-json/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ip-address/package.json              │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ip-regex/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/is-cidr/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/is-fullwidth-code-point/package.json │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/isexe/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/jackspeak/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/json-parse-even-better-errors/packa- │ node-pkg │        0        │    -    │
│ ge.json                                                                          │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/json-stringify-nice/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/jsonparse/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/just-diff-apply/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/just-diff/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmaccess/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmdiff/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmexec/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmfund/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmhook/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmorg/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmpack/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmpublish/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmsearch/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmteam/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmversion/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/lru-cache/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/make-fetch-happen/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minimatch/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-collect/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-fetch/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-flush/node_modules/minipas- │ node-pkg │        0        │    -    │
│ s/package.json                                                                   │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-flush/node_modules/yallist- │ node-pkg │        0        │    -    │
│ /package.json                                                                    │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-flush/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-pipeline/node_modules/mini- │ node-pkg │        0        │    -    │
│ pass/package.json                                                                │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-pipeline/node_modules/yall- │ node-pkg │        0        │    -    │
│ ist/package.json                                                                 │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-pipeline/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-sized/node_modules/minipas- │ node-pkg │        0        │    -    │
│ s/package.json                                                                   │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-sized/node_modules/yallist- │ node-pkg │        0        │    -    │
│ /package.json                                                                    │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-sized/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minizlib/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ms/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/mute-stream/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/negotiator/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/node-gyp/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/nopt/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/normalize-package-data/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-audit-report/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-bundled/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-install-checks/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-normalize-package-bin/package.j- │ node-pkg │        0        │    -    │
│ son                                                                              │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-package-arg/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-packlist/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-pick-manifest/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-profile/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-registry-fetch/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-user-validate/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/p-map/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/package-json-from-dist/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/pacote/package.json                  │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/parse-conflict-json/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/path-key/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/path-scurry/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/picomatch/package.json               │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/postcss-selector-parser/package.json │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/proc-log/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/proggy/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/promise-all-reject-late/package.json │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/promise-call-limit/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/promise-retry/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/promzard/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/qrcode-terminal/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/read-cmd-shim/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/read-package-json-fast/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/read/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/retry/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/safer-buffer/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/semver/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/shebang-command/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/shebang-regex/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/signal-exit/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/sigstore/package.json                │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/smart-buffer/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/socks-proxy-agent/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/socks/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/spdx-correct/node_modules/spdx-expr- │ node-pkg │        0        │    -    │
│ ession-parse/package.json                                                        │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/spdx-correct/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/spdx-exceptions/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/spdx-expression-parse/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/spdx-license-ids/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ssri/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/string-width-cjs/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/string-width/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/strip-ansi-cjs/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/strip-ansi/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/supports-color/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/tar/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/text-table/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/tiny-relative-date/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/tinyglobby/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/treeverse/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/tuf-js/node_modules/@tufjs/models/p- │ node-pkg │        0        │    -    │
│ ackage.json                                                                      │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/tuf-js/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/unique-filename/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/unique-slug/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/util-deprecate/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/validate-npm-package-license/node_m- │ node-pkg │        0        │    -    │
│ odules/spdx-expression-parse/package.json                                        │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/validate-npm-package-license/packag- │ node-pkg │        0        │    -    │
│ e.json                                                                           │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/validate-npm-package-name/package.j- │ node-pkg │        0        │    -    │
│ son                                                                              │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/walk-up-path/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/which/node_modules/isexe/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/which/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi-cjs/node_modules/ansi-sty- │ node-pkg │        0        │    -    │
│ les/package.json                                                                 │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi-cjs/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi/node_modules/ansi-regex/p- │ node-pkg │        0        │    -    │
│ ackage.json                                                                      │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi/node_modules/emoji-regex/- │ node-pkg │        0        │    -    │
│ package.json                                                                     │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi/node_modules/string-width- │ node-pkg │        0        │    -    │
│ /package.json                                                                    │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi/node_modules/strip-ansi/p- │ node-pkg │        0        │    -    │
│ ackage.json                                                                      │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/write-file-atomic/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/yallist/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/@zeit/schemas/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/ajv/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/ansi-align/node_modules/ansi-rege- │ node-pkg │        0        │    -    │
│ x/package.json                                                                   │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/ansi-align/node_modules/emoji-reg- │ node-pkg │        0        │    -    │
│ ex/package.json                                                                  │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/ansi-align/node_modules/string-wi- │ node-pkg │        0        │    -    │
│ dth/package.json                                                                 │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/ansi-align/node_modules/strip-ans- │ node-pkg │        0        │    -    │
│ i/package.json                                                                   │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/ansi-align/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/ansi-regex/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/ansi-styles/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/arg/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/balanced-match/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/boxen/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/brace-expansion/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/bytes/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/camelcase/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/chalk-template/node_modules/ansi-- │ node-pkg │        0        │    -    │
│ styles/package.json                                                              │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/chalk-template/node_modules/chalk- │ node-pkg │        0        │    -    │
│ /package.json                                                                    │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/chalk-template/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/chalk/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/cli-boxes/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/color-convert/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/color-name/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/compressible/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/compression/package.json           │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/concat-map/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/content-disposition/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/debug/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/deep-extend/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/eastasianwidth/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/emoji-regex/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/fast-deep-equal/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/fast-uri/benchmark/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/fast-uri/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/has-flag/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/ini/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/is-fullwidth-code-point/package.j- │ node-pkg │        0        │    -    │
│ son                                                                              │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/is-port-reachable/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/json-schema-traverse/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/mime-db/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/mime-types/node_modules/mime-db/p- │ node-pkg │        0        │    -    │
│ ackage.json                                                                      │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/mime-types/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/minimatch/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/minimist/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/ms/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/negotiator/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/on-headers/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/path-is-inside/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/path-to-regexp/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/range-parser/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/rc/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/registry-auth-token/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/registry-url/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/require-from-string/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/safe-buffer/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/serve-handler/node_modules/bytes/- │ node-pkg │        0        │    -    │
│ package.json                                                                     │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/serve-handler/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/string-width/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/strip-ansi/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/strip-json-comments/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/supports-color/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/tinyclip/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/type-fest/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/update-check/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/vary/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/widest-line/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/node_modules/wrap-ansi/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/serve/package.json                                    │ node-pkg │        0        │    -    │
└──────────────────────────────────────────────────────────────────────────────────┴──────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)


Node.js (node-pkg)

Total: 11 (HIGH: 11, CRITICAL: 0)

┌────────────────────────────────┬─────────────────┬──────────┬────────┬───────────────────┬──────────────────────────────┬─────────────────────────────────────────────────────────────┐
│            Library             │  Vulnerability  │ Severity │ Status │ Installed Version │        Fixed Version         │                            Title                            │
├────────────────────────────────┼─────────────────┼──────────┼────────┼───────────────────┼──────────────────────────────┼─────────────────────────────────────────────────────────────┤
│ brace-expansion (package.json) │ CVE-2026-102276 │ HIGH     │ fixed  │ 2.0.2             │ 5.0.10, 3.0.7, 2.1.5, 1.1.19 │ brace-expansion: brace-expansion: Denial of Service via     │
│                                │                 │          │        │                   │                              │ stack exhaustion from crafted brace patterns...             │
│                                │                 │          │        │                   │                              │ https://avd.aquasec.com/nvd/cve-2026-102276                 │
│                                ├─────────────────┤          │        │                   ├──────────────────────────────┼─────────────────────────────────────────────────────────────┤
│                                │ CVE-2026-102278 │          │        │                   │ 5.0.11, 3.0.8, 2.1.6, 1.1.20 │ brace-expansion: brace-expansion: Denial of Service via     │
│                                │                 │          │        │                   │                              │ uncontrolled recursion in nested brace patterns...          │
│                                │                 │          │        │                   │                              │ https://avd.aquasec.com/nvd/cve-2026-102278                 │
│                                ├─────────────────┤          │        │                   ├──────────────────────────────┼─────────────────────────────────────────────────────────────┤
│                                │ CVE-2026-13149  │          │        │                   │ 5.0.7, 1.1.16, 2.1.2         │ brace-expansion: Brace-expansion: Denial of Service due to  │
│                                │                 │          │        │                   │                              │ exponential-time complexity                                 │
│                                │                 │          │        │                   │                              │ https://avd.aquasec.com/nvd/cve-2026-13149                  │
│                                ├─────────────────┤          │        │                   ├──────────────────────────────┼─────────────────────────────────────────────────────────────┤
│                                │ CVE-2026-14257  │          │        │                   │ 5.0.8, 3.0.3, 2.1.3, 1.1.17  │ brace-expansion: Brace-expansion: Denial of Service via     │
│                                │                 │          │        │                   │                              │ memory exhaustion in expand() function                      │
│                                │                 │          │        │                   │                              │ https://avd.aquasec.com/nvd/cve-2026-14257                  │
│                                ├─────────────────┤          │        │                   ├──────────────────────────────┼─────────────────────────────────────────────────────────────┤
│                                │ CVE-2026-69152  │          │        │                   │ 1.1.18, 2.1.4, 3.0.6, 5.0.9  │ brace-expansion: DoS via unbounded intermediate arrays,     │
│                                │                 │          │        │                   │                              │ bypassing the CVE-2026-14257 mitigation                     │
│                                │                 │          │        │                   │                              │ https://avd.aquasec.com/nvd/cve-2026-69152                  │
├────────────────────────────────┼─────────────────┤          │        ├───────────────────┼──────────────────────────────┼─────────────────────────────────────────────────────────────┤
│ compression (package.json)     │ CVE-2026-87776  │          │        │ 1.8.1             │ 1.8.2                        │ compression: compression: Denial of Service via memory leak │
│                                │                 │          │        │                   │                              │ on premature response close...                              │
│                                │                 │          │        │                   │                              │ https://avd.aquasec.com/nvd/cve-2026-87776                  │
├────────────────────────────────┼─────────────────┤          │        ├───────────────────┼──────────────────────────────┼─────────────────────────────────────────────────────────────┤
│ ip-address (package.json)      │ CVE-2026-69192  │          │        │ 10.1.0            │ 10.3.1                       │ ip-address: ip-address: Inconsistent IP address parsing     │
│                                │                 │          │        │                   │                              │ leads to Server-Side Request Forgery (SSRF)...              │
│                                │                 │          │        │                   │                              │ https://avd.aquasec.com/nvd/cve-2026-69192                  │
├────────────────────────────────┼─────────────────┤          │        ├───────────────────┼──────────────────────────────┼─────────────────────────────────────────────────────────────┤
│ pacote (package.json)          │ CVE-2026-9496   │          │        │ 19.0.2            │ 21.5.1                       │ pacote: Pacote: Denial of Service via crafted spec.rawSpec  │
│                                │                 │          │        │                   │                              │ value in addGitSha function...                              │
│                                │                 │          │        │                   │                              │ https://avd.aquasec.com/nvd/cve-2026-9496                   │
│                                │                 │          │        ├───────────────────┤                              │                                                             │
│                                │                 │          │        │ 20.0.1            │                              │                                                             │
│                                │                 │          │        │                   │                              │                                                             │
│                                │                 │          │        │                   │                              │                                                             │
├────────────────────────────────┼─────────────────┤          │        ├───────────────────┼──────────────────────────────┼─────────────────────────────────────────────────────────────┤
│ picomatch (package.json)       │ CVE-2026-33671  │          │        │ 4.0.3             │ 4.0.4, 3.0.2, 2.3.2          │ picomatch: Picomatch: Regular Expression Denial of Service  │
│                                │                 │          │        │                   │                              │ via crafted extglob patterns                                │
│                                │                 │          │        │                   │                              │ https://avd.aquasec.com/nvd/cve-2026-33671                  │
├────────────────────────────────┼─────────────────┤          │        ├───────────────────┼──────────────────────────────┼─────────────────────────────────────────────────────────────┤
│ sigstore (package.json)        │ CVE-2026-48815  │          │        │ 3.1.0             │ 4.1.1                        │ sigstore: Sigstore: Unauthorized certificates accepted due  │
│                                │                 │          │        │                   │                              │ to ignored `certificateOIDs` verification option            │
│                                │                 │          │        │                   │                              │ https://avd.aquasec.com/nvd/cve-2026-48815                  │
└────────────────────────────────┴─────────────────┴──────────┴────────┴───────────────────┴──────────────────────────────┴─────────────────────────────────────────────────────────────┘
```
#### Dockle :

```bash
dockle quiz-docker:initial
WARN    - CIS-DI-0001: Create a user for the container
        * Last user should not be root
INFO    - CIS-DI-0005: Enable Content trust for Docker
        * export DOCKER_CONTENT_TRUST=1 before docker pull/build
INFO    - CIS-DI-0006: Add HEALTHCHECK instruction to the container image
        * not found HEALTHCHECK statement
INFO    - DKL-LI-0003: Only put necessary files
        * Suspicious directory : root/.npm 
```


## Rémédiation : 

1. Dockerfile multistage avec comme image finale `nginx/nginx-unprivileged`:

```bash

ARG NODE_VERSION=22.23.5-alpine3.24@sha256:1314fb30ec37592d5845e5a160d522c0e680972aa534549940b8d73bde88481a
ARG NGINX_VERSION=1.31.8-alpine3.24-slim@sha256:b19233c62309c8e620732244f9d0d0008678854cb9c6c47898817e8dec93a9e3

# Build stage
FROM node:${NODE_VERSION} AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

# Serve stage
FROM nginx/nginx-unprivileged:${NGINX_VERSION} AS serve

COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]

```

### Résultat : 

 1. Plus de vulnérabilités node.js :

``` bash
npm audit fix --force
npm warn using --force Recommended protections disabled.
npm warn audit Updating vite to 8.3.8, which is a SemVer major change.

added 10 packages, removed 3 packages, changed 1 package, and audited 17 packages in 18s

8 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities

 trivy image -q --severity HIGH,CRITICAL quiz-docker:optimized

Report Summary

┌───────────────────────────────────────┬────────┬─────────────────┬─────────┐
│                Target                 │  Type  │ Vulnerabilities │ Secrets │
├───────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ quiz-docker:optimized (alpine 3.24.4) │ alpine │        0        │    -    │
└───────────────────────────────────────┴────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)
```

 2. User non root : 

```bash
 exec -it quiz-docker-optimized sh
/ $ whoami
nginx
```
 3. Taille de l'image finale :

```bash
docker images quiz-docker:initial --format "{{.Size}}"
320MB
docker images quiz-docker:optimized --format "{{.Size}}"
21.7MB
```


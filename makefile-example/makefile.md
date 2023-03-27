## Golang Makefile 示例

### 获取Git仓库版本信息
```makefile
VERSION 	= $(strip $(subst v, , $(shell git describe --abbrev=0)))
HASH 		= $(shell git rev-parse --short HEAD)
TIME 		= $(shell git log --pretty=format:"%ad" --date=format:'%Y%m%d%H%M%S' $(HASH) -1)
```

### 注入版本信息
```makefile
GO_BUILD_FLAG := -ldflags="-w -s -X 'xxxx/buildinfo.build=${BUILD}' \
			        			 -X 'xxxx/buildinfo.version=${VERSION}' \
					 			 -X 'xxxx/buildinfo.devMode=false'"
```

###（AMD64）Linux交叉编译Windows
#### CGO
- 安装编译链
	```bash
	## CentOS
	yum install -y mingw-w64-tools mingw32-binutils

	## Ubuntu
	apt-get install -y mingw-w64
	```
- 设置Go编译选项
  ```makefile
  CGO_ENABLED=1 \
	CC=x86_64-w64-mingw32-gcc \
	CXX=x86_64-w64-mingw32-g++ \
	GOOS=windows \
	go build -a \
		-o "xxxx/" $(GO_BUILD_FLAGS) \
		main.go
  ```

#### Purn Go
```makefile
OS = darwin linux windows
ARCH = arm64 amd64

.PHONY: prod
prod:
	rm -rf cmd/xxxx/dist
	for arch in $(ARCH); do \
  		for os in $(OS); do \
			echo "release [$$os]-[$$arch] production"; \
			GOOS=$$os \
			GOARCH=$$arch \
			go build -a -tags prod -o "cmd/xxxx/dist/$$os/$$arch/" $(GO_BUILD_FLAGS) \
				cmd/xxxx/main.go; \
		done \
	done
```







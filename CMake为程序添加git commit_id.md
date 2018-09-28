# 背景说明
在我们开发C++程序的时候，经常不知道现在的bin文件对应于那一次git的提交，因此我们希望编译出来的bin文件在我们可以通过`--version`等参数，获取到对应git提交的commit id
# 获取分支名称和commit id
在git里面，我们可以通过以下的命令来获取当前目录的git分支名称和commit id
```shell
# 获取分支名称
git rev-parse --abbrev-ref HEAD
dev

# 获取commit id
git rev-parse HEAD
7b834083243a58d97b75f675a1098fba1f0998c8
```

# 实现的方法
通过上一步的git命令，我们可以得到当前目录的分支名称和commit id，那么我们可以在项目文件里面加入一个模板文件`version.h.template`，然后在cmake文件里面调用git的命令，修改模板文件里面的分支名称和commid id。最后就可以把分支名称和commit id编译到bin文件里面。
```cpp
// version.h.template
#ifndef VERSION_H
#define VERSION_H

#define GIT_BRANCH "@GIT_BRANCH@"
#define GIT_COMMIT_HASH "@GIT_COMMIT_HASH@"

#endif
```
```shell
# CMakeLists.txt

if(EXISTS "${CMAKE_SOURCE_DIR}/.git")
    execute_process(
        COMMAND git rev-parse --abbrev-ref HEAD
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        OUTPUT_VARIABLE GIT_BRANCH
        OUTPUT_STRIP_TRAILING_WHITESPACE
        )

    execute_process(
        COMMAND git rev-parse HEAD
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        OUTPUT_VARIABLE GIT_COMMIT_HASH
        OUTPUT_STRIP_TRAILING_WHITESPACE
        )
else(EXISTS "${CMAKE_SOURCE_DIR}/.git")
    set(GIT_BRANCH "")
    set(GIT_COMMIT_HASH "")
endif(EXISTS "${CMAKE_SOURCE_DIR}/.git")

message(STATUS "Git current branch: ${GIT_BRANCH}")
message(STATUS "Git commit hash: ${GIT_COMMIT_HASH}")

configure_file(
    ${CMAKE_SOURCE_DIR}/src/version/version.h.template
    ${CMAKE_SOURCE_DIR}/src/version/version.h
)
```
在我们执行`cmake .`命令的时候，cmake会执行上一步的git命令获取git的分支名称和commit id，并且会替换`version.h.template`里面的`GIT_BRANCH`和`GIT_COMMIT_HASH`变量，生成`version.h`文件。在我们的工程里面引入`version.h`头文件，然后就可以在工程里面得到git的分支名称和commit id。
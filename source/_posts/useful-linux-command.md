---
title: 实用 Linux 命令
date: 2021-06-28 11:03:14
categories: Linux
tags:
- linux
permalink: useful-linux-command
---
#### 1.替换字符串
```shell
grep --exclude=lang.json -rl "原文字" ./ | xargs gsed -i "s/原文字/新文字/g"
```
示例：
```shell
exec(
          `grep --exclude=lang.json -rl "t('${key}'" ./ | xargs gsed -i "s/t('${key}'/t('${value[0]}'/g"`,
          {
            cwd: path.join(process.cwd(), DEST),
          }
        );
```
<!--more-->

#### 2.统计字符串出现次数
```shell
grep --exclude=lang.json -rnw 目标字符串 . | wc -l
```
示例：
```shell
const count = execSync(`grep --exclude=lang.json -rnw ${key} . | wc -l`, { cwd: path.join(process.cwd(), DEST) });
```

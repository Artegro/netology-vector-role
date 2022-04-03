# Домашнее задание к занятию «2.4. Инструменты Git»

Для выполнения заданий в этом разделе давайте склонируем репозиторий с исходным кодом 
терраформа https://github.com/hashicorp/terraform 

В виде результата напишите текстом ответы на вопросы и каким образом эти ответы были получены. 

1. Найдите полный хеш и комментарий коммита, хеш которого начинается на `aefea`.
```bash
root@test:/home/art/devops-netology/teraform/terraform# git show aefea
commit aefead2207ef7e2aa5dc81a34aedf0cad4c32545
Author: Alisdair McDiarmid <alisdair@users.noreply.github.com>
Date:   Thu Jun 18 10:29:58 2020 -0400

    Update CHANGELOG.md

```
2. Какому тегу соответствует коммит `85024d3`?
```bash
git show 85024d3
commit 85024d3100126de36331c6982bfaac02cdab9e76 (tag: v0.12.23)
Соответсвтвено тег  v0.12.23
```
3. Сколько родителей у коммита `b8d720`? Напишите их хеши.
```bash
git log  --pretty=format:"%h - %P" | grep b8d720
38afbb34c - b8d720f8340221f2146e4e4870bf2ee0bc48f2d5
b8d720f83 - 56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b
# Узнаем колличество родителей , видим что их 2
# получаем хеши родителей
git show  b8d720^1
commit 56cd7859e05c36c06b56d013b55a252d0bb7e158
git show  b8d720^2
commit 9ea88f22fc6269854151c571162c5bcf958bee2b
```
4. Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами  v0.12.23 и v0.12.24.
```bash
git log  v0.12.23..v0.12.24 --oneline   
33ff1c03b (tag: v0.12.24) v0.12.24
b14b74c49 [Website] vmc provider links
3f235065b Update CHANGELOG.md
6ae64e247 registry: Fix panic when server is unreachable
5c619ca1b website: Remove links to the getting started guide's old location
06275647e Update CHANGELOG.md
d5f9411f5 command: Fix bug when using terraform login on Windows
4b6d06cc5 Update CHANGELOG.md
dd01a3507 Update CHANGELOG.md
225466bc3 Cleanup after v0.12.23 release
```
5. Найдите коммит в котором была создана функция `func providerSource`, ее определение в коде выглядит 
так `func providerSource(...)` (вместо троеточего перечислены аргументы).
```bash
# Находим комит где упоминается эта функция и поскольку комит 1 , то это указывает на появление той функции
git log -S"func providerSource(" --oneline
8c928e835 main: Consult local directories as potential mirrors of providers
# Далее роверяем что мы не ошиблись 
git show 8c928e835 | grep "func providerSource"
+func providerSource(services *disco.Disco) getproviders.Source {
```
6. Найдите все коммиты в которых была изменена функция `globalPluginDirs`.
```bash
# Находим всме коммиты  где упоминается эта функция
git log -S"globalPluginDirs" --oneline
35a058fb3 main: configure credentials from the CLI config file
c0b176109 prevent log output during init
8364383c3 Push plugin discovery down into command package
# Впринциеп это все . но на всякий проеряем что там небыло удаления этой функции , смотрим тапорно глазками.. 
git show c0b176109 |grep globalPluginDirs
+               // FIXME: homeDir gets called from globalPluginDirs during init, before
root@test:/home/art/devops-netology/teraform/terraform# git show 35a058fb3 |grep globalPluginDirs
+               available := pluginDiscovery.FindPlugins("credentials", globalPluginDirs())
root@test:/home/art/devops-netology/teraform/terraform# git show 8364383c3 |grep globalPluginDirs
+               GlobalPluginDirs: globalPluginDirs(),
+// globalPluginDirs returns directories that should be searched for
+func globalPluginDirs() []string {
```
8. Кто автор функции `synchronizedWriters`? 
```bash
# Запускаем мой уже любимый git log и видим, что автор первого коммит где появляется упоминание этой функции Martin Atkins
git log -S"synchronizedWrite" --pretty=format:"%H - %an"
bdfea50cc85161dea41be0fe3381fd98731ff786 - James Bardin
fd4f7eb0b935e5a838810564fd549afe710ae19a - James Bardin
5ac311e2a91e381e2f52234668b49ba670aa0fe5 - Martin Atkins
```

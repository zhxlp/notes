# debian 保持软件包一致

- 导出已安装软件包列表

  ```bash
  dpkg --get-selections > list.txt
  apt list --installed | awk -F"/" '$2!="" {printf "%s ",$1}' > installed.txt
  ```

- 安装软件包，并保持一致

  ```bash
  apt install -y --no-install-recommends < installed.txt
  dpkg --clear-selections
  dpkg --set-selections <list.txt
  apt-get dselect-upgrade
  ```

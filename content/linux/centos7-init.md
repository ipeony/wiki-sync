---
title: Centos7 初始化ansible脚本
date: 2017-06-20 16:36:39
tag: ansible, linux, centos
---
[TOC]

### ansible脚本
[点击此处下载](../../attach/centos7-init.yml)
```yml
---
- hosts: vps
  remote_user: root

  vars_prompt:
    - name: "hostname"
      prompt: "Input new hostname"
      private: no

  tasks:
  - name: set hostname
    hostname:
      name: "{{ hostname }}"
  - name: config /etc/hosts
    lineinfile:
      path: /etc/hosts
      line: '127.0.0.1 {{ hostname }}'
  - name: create nopasswd user
    user:
      name: dongfg
  - name: add ssh public key to user
    authorized_key:
      user: dongfg
      state: present
      key: "{{ lookup('file', '/home/dongfg/.ssh/vps.pub') }}"
  - name: grant sudo privilege with no password
    copy:
      content: "dongfg ALL=(ALL) NOPASSWD:ALL"
      dest: /etc/sudoers.d/90-init-user
  - name: Add repository
    yum_repository:
      name: epel
      description: EPEL YUM repo
      baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/
      gpgcheck: no
  - name: install basic packages
    yum: 
      name: "{{ item }}"
      state: installed
    with_items:
      - git
      - zsh
      - htop
      - tcping
  - name: install oh my zsh
    command: "{{ item }}"
    become: true
    become_user: dongfg
    with_items:
      - rm -rf ~/.oh-my-zsh
      - git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
      - cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
  - name: change user shell
    user:
      name: dongfg
      shell: /bin/zsh
```
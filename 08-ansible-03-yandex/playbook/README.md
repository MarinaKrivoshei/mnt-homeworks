### Playbook
___
Дополняем playbook данными для NGINX.
```
- name: Install nginx
  hosts: lighthouse
  handlers:
    - name: start-nginx
      become: true
      command: nginx
    - name: reload-nginx
      become: true
      command: nginx reload
  tasks:
    - name: NGINX | Install epel-release
      become: true
      ansible.builtin.yum:
        name: epel-release
        state: present
      tags: nginx
    - name: Install NGINX
      become: true
      ansible.builtin.yum:
        name: nginx
        state: present
      notify: start-nginx
      tags: nginx 
 ```
Создается конфиг NGINX
 ```
    - name: Create config NGINX
      become: true
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        mode: 0644
      notify: reload-nginx
      tags: nginx
  ```


```
- name: Install lighthouse
  hosts: lighthouse
  handlers:
    - name: reload-nginx
      become: true
      command: nginx -s reload
 ```
Прописывается pre_tasks, в которой устанавливаем GIT.
```
  pre_tasks:
    - name: Install git
      become: true
      ansible.builtin.yum:
        name: git
        state: present
 ```
И указываем tasks, который скачивает lighthouse с git. Создает конфиг lighthouse и делает перезагрузку nginx
```
  tasks:
    - name: Copy Lighthouse from git
      git:
        repo: "{{ lighthouse_vcs }}"
        version: master
        dest: "{{ lighthouse_location_dir  }}"
        clone: yes
        update: yes
      tags: lighthouse
    - name: Create lighthouse config
      become: true
      template:
        src: templates/lighthouse.conf.j2
        dest: /etc/nginx/conf.d/default.conf
        mode: 0644
      notify: reload-nginx
      tags: lighthouse
  ```





![img.png](img.png)

## การทำงาน

- การทำงานหลักเป็นไปตามบทความนี้ครับ: [Simple automated GIT Deployment using GIT Hooks](https://gist.github.com/noelboss/3fe13927025b89757f8fb12e9066f2fa)
- ก่อนการทำงาน playbook นี้จะ allow SSH port ที่ remote host ก่อนด้วย `ufw` และจะ limit กลับในภายหลัง
- รายชื่อ hosts หรือ IP address ทั้งหมดที่ระบุใน `hosts` file หรือ `-i` flag ของ `ansible-playbook` command จะถูกใช้ เนื่องจาก set ค่า `hosts` ไว้เป็น `all` ครับ
- หลังจาก set up remote host(s) เสร็จแล้ว playbook นี้จะเพิ่ม remote repository ไปยัง local repository (`git remote add ...`) **ก็ต่อเมื่อ**ได้รับ arguments `local_work_tree` **และ** กรณีที่ run on multiple hosts **ต้องไม่** ใส่ argument `remote_name` ถ้าจะใส่ `remote_name` ต้องมี remote host เดียวเท่านั้นครับ

## Todos

- [ ] support multiple remotes with same name using `git remote set-url --add --push <name> <url>`. See: [https://stackoverflow.com/questions/14290113/git-pushing-code-to-two-remotes](https://stackoverflow.com/questions/14290113/git-pushing-code-to-two-remotes)

## Issues

- [x] ~~Incorrect work tree and repo directory path. Change it to be an absolute path.~~

---

## ก่อนใช้ควรจะ...

- ใช้ Git เป็นไม่มากก็น้อยครับ
- อ่านบทความนี้ก่อนครับ: [Simple automated GIT Deployment using GIT Hooks](https://gist.github.com/noelboss/3fe13927025b89757f8fb12e9066f2fa)
- ใช้งาน [Ansible](https://docs.ansible.com/ansible/latest/index.html) เป็น
- มี `git` และ `ansible-playbook` command ให้ใช้ในเครื่อง host ครับ

---

## วิธีใช้

### แบบบรรทัดเดียวจบ

แต่เขียนหลายบรรทัดเพื่อให้อ่านง่ายนะครับ

```shell
ansible-playbook -i '<remote-host>,' playbook.yml \
    -e repo_name=<repository-name> \
    -e u=<username> \
    -e local_work_tree=<path/to/local/work/tree> \
    -e remote_name=<custom-origin-name>
```

ตัวอย่างเช่น:

```shell
ansible-playbook -i 'example.com,' playbook.yml \
    -e repo_name=ansible-git-auto-deploy \
    -e u=nawawishkid \
    -e local_work_tree=. \
    -e remote_name=staging
```

### แบบเพิ่ม remote ไปที่ local เอง

1.

```shell
ansible-playbook -i '<remote-host>,' playbook.yml \
    -e repo_name=<repo_name> \
    -e u=<username>
```

2.

```shell
git remote add <remote-name> <username>@<remote-host>:.agad-git/<repo_name>.git
```

---

## Arguments

| ชื่อ              | จำเป็นไหม | ค่าเริ่มต้น                             | คำอธิบาย                                                                                                                                                                                                    |
| ----------------- | --------- | --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `repo_name`       | ใช่       | `undefined`                             | ชื่อของ repository ที่จะถูกใช้เป็นชื่อ directory ของ work tree และ bare Git repository บน remote host ครับ                                                                                                  |
| `u`               | ใช่       | `undefined`                             | ชื่อของ user ที่จะใช้ access remote host                                                                                                                                                                    |
| `local_work_tree` | ไม่       | `undefined`                             | path ของ work tree directory บนเครื่อง localhost จะเป็น absolute หรือ relative path ก็ได้ ใช้เมื่อต้องการให้เพิ่ม remote repository ไปยัง local repository หลังจากที่สร้าง remote repository เสร็จครับ      |
| `remote_name`     | ไม่       | ชื่อ remote host (`inventory_hostname`) | ชื่อของ remote repository ที่จะถูกใช้ใน `git remote add <remote_name> ...` เหตุผลที่จะใช้ก็แบบเดียวกับ `local_work_tree` ครับ แต่ต้อง run บน remote host เดียวเท่านั้น **อย่าใช้**ถ้า run on multiple hosts |
| `work_tree_dirs`  | ไม่       | `.agad-work-tree`                       | Top directory ที่บรรดา work tree directories ทั้งหลายจะมารวมกันอยู่ในนี้                                                                                                                                    |
| `git_dirs`        | ไม่       | `.agad-git`                             | Top directory ที่บรรดา bare Git repositories จะมารวมกันอยู่ในนี้                                                                                                                                            |
| `branch`          | ไม่       | `master`                                | Branch ของ repository ที่จะถูก check out บน remote host                                                                                                                                                     |
| `post_checkout`   | ไม่       | `undefined`                             | path ของ shell script file ที่ content จะถูก append ไปยัง post-receive hook file เพื่อ run หลังจาก checkout files ไปแล้ว                                                                                    |

# Gitblog main.py

## 内容来源

- [Deepseek：#Gitblog main.py 代码解析](https://chat.deepseek.com/share/0k49t8521q3gk2acx9)
- [The requested URL returned error: 403](https://github.com/shenjuexiao/giscafer/issues/1)

## main.py

```python
# main.py
# -*- coding: utf-8 -*-
import argparse
import os
import re

import markdown
from feedgen.feed import FeedGenerator
from github import Github
from lxml.etree import CDATA
from marko.ext.gfm import gfm as marko

MD_HEAD = """## [Gitblog](https://yihong0618.github.io/gitblog/)
My personal blog([About Me](https://github.com/yihong0618/gitblog/issues/282)) using issues and GitHub Actions (随意转载，无需署名)
[Things I like](https://github.com/yihong0618/gitblog/issues/311)
![image](https://github.com/user-attachments/assets/a168bf11-661e-4566-b042-7fc9544de528)
[RSS Feed](https://raw.githubusercontent.com/{repo_name}/master/feed.xml)
"""

BACKUP_DIR = "BACKUP"
ANCHOR_NUMBER = 5
TOP_ISSUES_LABELS = ["Top"]
TODO_ISSUES_LABELS = ["TODO"]
FRIENDS_LABELS = ["Friends"]
ABOUT_LABELS = ["About"]
THINGS_LABELS = ["Things"]
IGNORE_LABELS = (
    FRIENDS_LABELS
    + TOP_ISSUES_LABELS
    + TODO_ISSUES_LABELS
    + ABOUT_LABELS
    + THINGS_LABELS
)

FRIENDS_TABLE_HEAD = "| Name | Link | Desc | \n | ---- | ---- | ---- |\n"
FRIENDS_TABLE_TEMPLATE = "| {name} | {link} | {desc} |\n"
FRIENDS_INFO_DICT = {
    "名字": "",
    "链接": "",
    "描述": "",
}


def get_me(user):
    return user.get_user().login


def is_me(issue, me):
    return issue.user.login == me


def is_hearted_by_me(comment, me):
    reactions = list(comment.get_reactions())
    for r in reactions:
        if r.content == "heart" and r.user.login == me:
            return True
    return False


def _make_friend_table_string(s):
    info_dict = FRIENDS_INFO_DICT.copy()
    try:
        string_list = s.splitlines()
        # drop empty line
        string_list = [l for l in string_list if l and not l.isspace()]
        for l in string_list:
            string_info_list = re.split("：", l)
            if len(string_info_list) < 2:
                continue
            info_dict[string_info_list[0]] = string_info_list[1]
        return FRIENDS_TABLE_TEMPLATE.format(
            name=info_dict["名字"], link=info_dict["链接"], desc=info_dict["描述"]
        )
    except Exception as e:
        print(str(e))
        return


# help to covert xml vaild string
def _valid_xml_char_ordinal(c):
    codepoint = ord(c)
    # conditions ordered by presumed frequency
    return (
        0x20 <= codepoint <= 0xD7FF
        or codepoint in (0x9, 0xA, 0xD)
        or 0xE000 <= codepoint <= 0xFFFD
        or 0x10000 <= codepoint <= 0x10FFFF
    )


def format_time(time):
    return str(time)[:10]


def login(token):
    return Github(token)


def get_repo(user: Github, repo: str):
    return user.get_repo(repo)


def parse_TODO(issue):
    body = issue.body.splitlines()
    todo_undone = [l for l in body if l.startswith("- [ ] ")]
    todo_done = [l for l in body if l.startswith("- [x] ")]
    # just add info all done
    if not todo_undone:
        return f"[{issue.title}]({issue.html_url}) all done", []
    return (
        f"[{issue.title}]({issue.html_url})--{len(todo_undone)} jobs to do--{len(todo_done)} jobs done",
        todo_done + todo_undone,
    )


def get_top_issues(repo):
    return repo.get_issues(labels=TOP_ISSUES_LABELS)


def get_todo_issues(repo):
    return repo.get_issues(labels=TODO_ISSUES_LABELS)


def get_repo_labels(repo):
    return [l for l in repo.get_labels()]


def get_issues_from_label(repo, label):
    return repo.get_issues(labels=(label,))


def add_issue_info(issue, md):
    time = format_time(issue.created_at)
    md.write(f"- [{issue.title}]({issue.html_url})--{time}\n")


def add_md_todo(repo, md, me):
    todo_issues = list(get_todo_issues(repo))
    if not TODO_ISSUES_LABELS or not todo_issues:
        return
    with open(md, "a+", encoding="utf-8") as md:
        md.write("## TODO\n")
        for issue in todo_issues:
            if is_me(issue, me):
                todo_title, todo_list = parse_TODO(issue)
                md.write("TODO list from " + todo_title + "\n")
                for t in todo_list:
                    md.write(t + "\n")
                # new line
                md.write("\n")


def add_md_top(repo, md, me):
    top_issues = list(get_top_issues(repo))
    if not TOP_ISSUES_LABELS or not top_issues:
        return
    with open(md, "a+", encoding="utf-8") as md:
        md.write("## 置顶文章\n")
        for issue in top_issues:
            if is_me(issue, me):
                add_issue_info(issue, md)


def add_md_firends(repo, md, me):

    s = FRIENDS_TABLE_HEAD
    friends_issues = list(repo.get_issues(labels=FRIENDS_LABELS))
    if not FRIENDS_LABELS or not friends_issues:
        return
    friends_issue_number = friends_issues[0].number
    for issue in friends_issues:
        for comment in issue.get_comments():
            if is_hearted_by_me(comment, me):
                try:
                    s += _make_friend_table_string(comment.body or "")
                except Exception as e:
                    print(str(e))
                    pass
    s = markdown.markdown(s, output_format="html", extensions=["extra"])
    with open(md, "a+", encoding="utf-8") as md:
        md.write(
            f"## [友情链接](https://github.com/{str(me)}/gitblog/issues/{friends_issue_number})\n"
        )
        md.write("<details><summary>显示</summary>\n")
        md.write(s)
        md.write("</details>\n")
        md.write("\n\n")


def add_md_recent(repo, md, me, limit=5):
    count = 0
    with open(md, "a+", encoding="utf-8") as md:
        # one the issue that only one issue and delete (pyGitHub raise an exception)
        try:
            md.write("## 最近更新\n")
            for issue in repo.get_issues(sort="created", direction="desc"):
                if is_me(issue, me):
                    add_issue_info(issue, md)
                    count += 1
                    if count >= limit:
                        break
        except Exception as e:
            print(str(e))


def add_md_header(md, repo_name):
    with open(md, "w", encoding="utf-8") as md:
        md.write(MD_HEAD.format(repo_name=repo_name))
        md.write("\n")


def add_md_label(repo, md, me):
    labels = get_repo_labels(repo)

    # sort lables by description info if it exists, otherwise sort by name,
    # for example, we can let the description start with a number (1#Java, 2#Docker, 3#K8s, etc.)
    labels = sorted(
        labels,
        key=lambda x: (
            x.description is None,
            x.description == "",
            x.description,
            x.name,
        ),
    )

    with open(md, "a+", encoding="utf-8") as md:
        for label in labels:
            # we don't need add top label again
            if label.name in IGNORE_LABELS:
                continue

            issues = get_issues_from_label(repo, label)
            issues = list(sorted(issues, key=lambda x: x.created_at, reverse=True))
            if len(issues) != 0:
                md.write("## " + label.name + "\n\n")
            i = 0
            for issue in issues:
                if not issue:
                    continue
                if is_me(issue, me):
                    if i == ANCHOR_NUMBER:
                        md.write("<details><summary>显示更多</summary>\n")
                        md.write("\n")
                    add_issue_info(issue, md)
                    i += 1
            if i > ANCHOR_NUMBER:
                md.write("</details>\n")
                md.write("\n")


def get_to_generate_issues(repo, dir_name, issue_number=None):
    md_files = os.listdir(dir_name)
    generated_issues_numbers = [
        int(i.split("_")[0]) for i in md_files if i.split("_")[0].isdigit()
    ]
    to_generate_issues = [
        i
        for i in list(repo.get_issues())
        if int(i.number) not in generated_issues_numbers
    ]
    if issue_number:
        to_generate_issues.append(repo.get_issue(int(issue_number)))
    return to_generate_issues


def generate_rss_feed(repo, filename, me):
    generator = FeedGenerator()
    generator.id(repo.html_url)
    generator.title(f"RSS feed of {repo.owner.login}'s {repo.name}")
    generator.author(
        {"name": os.getenv("GITHUB_NAME"), "email": os.getenv("GITHUB_EMAIL")}
    )
    generator.link(href=repo.html_url)
    generator.link(
        href=f"https://raw.githubusercontent.com/{repo.full_name}/master/{filename}",
        rel="self",
    )
    for issue in repo.get_issues():
        if not issue.body or not is_me(issue, me) or issue.pull_request:
            continue
        item = generator.add_entry(order="append")
        item.id(issue.html_url)
        item.link(href=issue.html_url)
        item.title(issue.title)
        item.published(issue.created_at.strftime("%Y-%m-%dT%H:%M:%SZ"))
        for label in issue.labels:
            item.category({"term": label.name})
        body = "".join(c for c in issue.body if _valid_xml_char_ordinal(c))
        item.content(CDATA(marko.convert(body)), type="html")
    generator.atom_file(filename)


def main(token, repo_name, issue_number=None, dir_name=BACKUP_DIR):
    user = login(token)
    me = get_me(user)
    repo = get_repo(user, repo_name)
    # add to readme one by one, change order here
    add_md_header("README.md", repo_name)
    for func in [add_md_firends, add_md_top, add_md_recent, add_md_label, add_md_todo]:
        func(repo, "README.md", me)

    generate_rss_feed(repo, "feed.xml", me)
    to_generate_issues = get_to_generate_issues(repo, dir_name, issue_number)

    # save md files to backup folder
    for issue in to_generate_issues:
        save_issue(issue, me, dir_name)


def save_issue(issue, me, dir_name=BACKUP_DIR):
    md_name = os.path.join(
        dir_name, f"{issue.number}_{issue.title.replace('/', '-').replace(' ', '.')}.md"
    )
    with open(md_name, "w") as f:
        f.write(f"# [{issue.title}]({issue.html_url})\n\n")
        f.write(issue.body or "")
        if issue.comments:
            for c in issue.get_comments():
                if is_me(c, me):
                    f.write("\n\n---\n\n")
                    f.write(c.body or "")


if __name__ == "__main__":
    if not os.path.exists(BACKUP_DIR):
        os.mkdir(BACKUP_DIR)
    parser = argparse.ArgumentParser()
    parser.add_argument("github_token", help="github_token")
    parser.add_argument("repo_name", help="repo_name")
    parser.add_argument(
        "--issue_number", help="issue_number", default=None, required=False
    )
    options = parser.parse_args()
    main(options.github_token, options.repo_name, options.issue_number)
```


这是一个使用GitHub Issues作为博客系统的Python脚本，名为Gitblog。它通过GitHub API将issues转换为静态博客内容。我来详细解析它的功能：

## 主要功能

### 1. **README.md生成**
脚本会生成一个README.md文件，包含以下部分：
- **置顶文章**：带有"Top"标签的issues
- **最近更新**：最新的5篇文章
- **按标签分类**：其他所有issues按标签分组显示
- **友情链接**：通过"Friends"标签的issue评论区管理
- **TODO列表**：带有"TODO"标签的issue中的任务列表

### 2. **RSS Feed生成**
自动生成`feed.xml`文件，支持订阅博客更新。

### 3. **文章备份**
将所有文章备份到`BACKUP`目录，以`{issue编号}_{标题}.md`格式保存。

## 代码结构

### 核心函数：

| 函数 | 作用 |
|------|------|
| `login()` | 使用GitHub token登录 |
| `get_repo()` | 获取指定的仓库 |
| `is_me()` | 判断issue是否由自己创建 |
| `is_hearted_by_me()` | 判断评论是否被自己点赞（❤️） |
| `parse_TODO()` | 解析TODO列表，统计完成/未完成项 |
| `generate_rss_feed()` | 生成RSS订阅文件 |
| `save_issue()` | 备份单个issue到本地 |

### 特殊标签处理：
- **Top**：置顶文章
- **TODO**：任务列表
- **Friends**：友情链接
- **About/Things**：特殊页面（被忽略不显示在普通标签中）

## 使用方式

```bash
python main.py <github_token> <repo_name> [--issue_number <number>]
```

### 参数说明：
- `github_token`：GitHub个人访问令牌
- `repo_name`：仓库名称（格式：`username/repo`）
- `--issue_number`：可选，指定备份特定文章

## 技术特点

1. **依赖库**：
   - `feedgen`：生成RSS feed
   - `github`：GitHub API客户端（PyGithub）
   - `markdown`/`marko`：Markdown解析
   - `lxml`：XML处理

2. **自动化**：可通过GitHub Actions定时运行，自动更新博客

3. **数据持久化**：通过BACKUP目录缓存，避免重复处理

4. **评论支持**：可以保留issue的评论内容

这个项目特别适合开发者用GitHub Issues管理个人博客，无需额外服务器，完全利用GitHub平台。
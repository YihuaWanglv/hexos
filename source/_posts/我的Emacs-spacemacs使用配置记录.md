---
title: 我的Emacs-spacemacs使用配置记录
date: 2018-09-13 23:55:55
tags:
---


此文章为我的Emacs/spacemacs配置和使用记录。持续更新中...

其中记录分为2类，如果是spacemacs专用配置的，会记录于spacemacs类目下，否则默认是记录于Emacs类别下。


# 1. emacs

## 1.1 日常高频操作

- 打开一个新的系统窗口
```
M-x make-frame <Return>
```

- 指定一个文件出现在下方的窗格中，同时光标也跳到了那里

输入 C-x 4 C-f，紧跟着输入一个文件名，再用 <Return> 结束。

- 将光标转移到其他的窗格。

输入 C-x o（“o”指的是“其它（other）”），

- 滚动下方的窗格。
```
C-M-v 
```

- 复制/剪切和粘贴
```
C-w 剪切
M-w 复制
C-y 粘贴
```

- 搜索和替换

C-s 向后搜索，然后输入需要搜索的文字
M-% 查找和替换，输入M-%后，先输入要被替换的文字，Ret，再输入要替换的新文字

- 复制一行
```
Ctrl-a 光标到行首
Ctrl-Shift-Space 设置标记
Ctrl-e 光标到行尾。如此这一行就被选为激活的区域了
Alt-w 复制当前激活的区域
```

- 修改配置文件后，重新加载配置

C-x C-e ;; current line
M-x eval-region ;; region
M-x eval-buffer ;; whole buffer
M-x load-file ~/.emacs.d/init.el


## 1.2 常用配置

- 定义快捷键快速打开配置文件
(defun open-my-init-file()
  (interactive)
  (find-file"C:/Users/iyihua/AppData/Roaming/.emacs"))
  (global-set-key(kbd "<f2>") 'open-my-init-file)

- 设置在emacs中打开git bash
;;set git base as emacs shell
(prefer-coding-system 'utf-8)
(defun iyihua/bash() 
  (interactive)
  (let ((explicit-shell-file-name "D:/tools/git/bin/bash"))
  (call-interactively 'shell)))
(prefer-coding-system 'utf-8)
(defun iyihua/git-bash() 
  (interactive)
  (let ((explicit-shell-file-name "D:/tools/git/git-bash"))
  (call-interactively 'shell)))

- 添加packages源

```
https://mirror.tuna.tsinghua.edu.cn/help/elpa/

Installing
To use the MELPA repository, you'll need an Emacs with package.el. Emacs 24 has package.el bundled with it, and there's also a version you can use with Emacs 23.

Enable installation of packages from MELPA by adding an entry to package-archives after (require 'package) and before the call to package-initialize in your init.el or .emacs file:

(require 'package)
(let* ((no-ssl (and (memq system-type '(windows-nt ms-dos))
                    (not (gnutls-available-p))))
       (proto (if no-ssl "http" "https")))
  ;; Comment/uncomment these two lines to enable/disable MELPA and MELPA Stable as desired
  (add-to-list 'package-archives (cons "melpa" (concat proto "://melpa.org/packages/")) t)
  ;;(add-to-list 'package-archives (cons "melpa-stable" (concat proto "://stable.melpa.org/packages/")) t)
  (when (< emacs-major-version 24)
    ;; For important compatibility libraries like cl-lib
    (add-to-list 'package-archives '("gnu" . (concat proto "://elpa.gnu.org/packages/")))))
(package-initialize)
To use the stable package repository instead of the default “bleeding-edge” repository, use this instead of "melpa":

(add-to-list 'package-archives
             '("melpa-stable" . "https://stable.melpa.org/packages/") t)
```

- 设置自动保存
```
;;; packages.el start
(require 'auto-save)
(defconst iyihua-packages
  '(auto-save)
)

(defun iyihua/init-auto-save()
  (use-package auto-save
    :init
  (auto-save-enable)
  (setq auto-save-slient t)))

;;; packages.el ends here
```

- 安装主题
```
Install manually
Add the emacs theme files to ~/.emacs.d/themes.

To load a theme add the following to your init.el

(add-to-list 'custom-theme-load-path "~/.emacs.d/themes")
(load-theme 'dracula t)
```

- 设置默认字体

前提是系统已下载安装对应字体

```
(custom-set-faces
 '(default ((t (:family "Source Code Pro" :foundry "outline" :slant normal :weight normal :height 98 :width normal)))))
```

- 显示隐藏menu bar

```
;;show/hide menu bar
(menu-bar-mode -1)
(global-set-key [f9] 'toggle-menu-bar-mode-from-frame)
```

# 2. spacemacs
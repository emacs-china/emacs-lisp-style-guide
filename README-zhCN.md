# Emacs Lisp 风格指南

> 榜样很重要 <br>
> ——墨菲警官《机器战警》

这份 Emacs Lisp 风格指南向你推荐实际使用中的最佳实践，Emacs Lisp 程序员如何写出可被别的 Emacs Lisp 程序员维护的代码。我们只说实际使用中的用法。指南再好，但里面说的过于理想化结果大家拒绝使用或者可能根本没人用，又有何意义。

本指南依照相关规则分成数个小节。我尽力在规则后面说明理由（如果省略了说明，那是因为其理由显而易见）。

规则不是我凭空想出来的——绝大部分来自我作为从业多年的职业软件工程师的经验，从 Emacs Lisp 社区成员得到的反馈及建议，和几个评价甚高的 Emacs Lisp 编程资源，像 ["GNU Emacs Lisp Reference Manual"](https://www.gnu.org/software/emacs/manual/elisp.html)。

本指南仍在完善中——有些章节还没写，另一些则不完整，某些规则缺乏实例，某些例子也不够清楚。到时候都会解决的——放心吧。

请注意，Emacs 开发者也维护着一份 [coding conventions and tips](http://www.gnu.org/software/emacs/manual/html_node/elisp/Tips.html#Tips)。

你可以使用 [Transmuter](https://github.com/TechnoGate/transmuter) 生成本指南的 PDF 或 HTML 版本。

## 目录

* [源代码排版和组织](#源代码排版和组织)
* [语法](#语法)
* [命名](#命名)
* [字符串](#字符串)
* [Macros](#macros)
    * [Macro Declarations](#macro-declarations)
* [Anonymous (lambda) functions](#anonymous-lambda-functions)
* [Loading and Autoloading](#loading-and-autoloading)
* [注释](#注释)
    * [Comment Annotations](#comment-annotations)
    * [Docstrings](#docstrings)
* [Existential](#existential)

## 源代码排版和组织

> 所有风格都又丑又难读，自己的除外。几乎人人都这样想。把“自己的除外”拿掉，他们或许是对的...<br>
> ——Jerry Coffin（论缩排）

以下指定的缩排规则 Emacs 已经默认采用了，当你输入 `<tab>` 时， 缩排总是正确的。

* 使用**空格**缩进。不要使用硬 tab。

* 对于普通的函数，垂直对齐参数。

    ```el
    ;; 好
    (format "%s %d"
            something
            something-else)

    ;; 差
    (format "%s %d"
      something
      something-else)
    ```

* 如果第一个参数在新的一行上，与函数名对齐。

    ```el
    ;; 好
    (format
     "%s %d"
     something
     something-else)

    ;; 差
    (format
      "%s %d"
      something
      something-else)
    ```

* 有些特殊 form，它们接受一个或多个_"special"_参数，紧接着一个_"body"_（一个任意长度的参数，但只有最后的返回值有意义），比如 `if`、`let`、`with-current-buffer` 等等。_"special"_参数应该与 form 名放在同一行或用四个空格缩排。_"body"_参数应该用两个空格缩排。

    ```el
    ;; 好
    (when something
      (something-else))

    ;; 差 -  body 参数用了四个空格缩排
    (when something
        (something-else))

    ;; 差 - 像普通的函数一样对齐了
    (when
     something
     (something-else))
    ```

* `if` form 的 "if" 分支是一个_"special"_参数，应该用四个空格缩排。

    ```el
    ;; 好
    (if something
        if-clause
      else-clause)

    ;; 差
    (if something
      if-clause
      else-clause)
    ```

* 竖直对齐 `let` 的绑定。

    ```el
    ;; 好
    (let ((thing1 "some stuff")
          (thing2 "other stuff")
      ...)

    ;; 差
    (let ((thing1 "some stuff")
      (thing2 "other stuff"))
      ...)
    ```
* 使用 Unix 风格的换行符。（BSD/Solaris/Linux/OSX 的用户不用担心，Windows 用户要格外小心。）
    * 如果你使用 Git ，可用下面这个配置，来保护你的项目不被 Windows 的换行符干扰：

    ```bash
    $ git config --global core.autocrlf true
    ```

* 如果有 text 位于开括号（`(`，`{`和`[`）之前，或位于闭括号（`)`，`}`和`]`）之后，用一个空格隔开  text 和括号。相反地，不要在开括号之后和 text 之前，或 text 之前和闭括号之前，留任何空白字符。

* 把所有的尾缀括号放在同一行上，而不是另起一行。

    ```el
    ;; 好 ; 同一行
    (when something
      (something-else))

    ;; 差 ; 另起一行
    (when something
      (something-else)
    )
    ```

* 使用空行来区分 top-level forms。

    ```el
    ;; 好
    (defvar x ...)

    (defun foo ...)

    ;; 差
    (defvar x ...)
    (defun foo ...)
    ```

    一个例外需要把相关的 `def`s 放在一起时。

    ```el
    ;; 好
    (defconst min-rows 10)
    (defconst max-rows 20)
    (defconst min-cols 15)
    (defconst max-cols 30)
    ```

* 不要在函数或者宏定义的中间使用空白行。一个例外是要提示成对出现的构造语句，比如在 `let` 和 `cond` 中出现的。

* 每一行限制在 80 个字符内。

* 避免行尾空格。

* 避免参数列表中多于三个或四个占位的参数。

* 总是开启 lexical scoping。且必须在作为一个 file local variable 放在文件首行。

    ```el
    ;;; -*- lexical-binding: t; -*-
    ```

## 语法

* 不要把 `if` form 的 "else" 分支包括在 `pron` 中（默认已经包括了）。

    ```el
    ;; 好
    (if something
        if-clause
      (something)
      (something-else))

    ;; 差
    (if something
        if-clause
      (progn
        (something)
        (something-else)))
    ```

* 用 `when` 而不是 `(if ... (progn ...)`。

    ```el
    ;; 好
    (when pred
      (foo)
      (bar))

    ;; 差
    (if pred
      (progn
        (foo)
        (bar)))
    ```

* 用 `unless` 而不是 `(when (not ...) ...)`。

    ```el
    ;; 好
    (unless pred
      (foo)
      (bar))

    ;; 差
    (when (not pred)
      (foo)
      (bar))
    ```

* 作比较时，留意 `<`，`>` 等函数可以接受任意数量的参数，从 Emacs 24.4 起。

    ```el
    ;; 更好
    (< 5 x 10)

    ;; 老的方法
    (and (> x 5) (< x 10))
    ```

* 用 `t` 作为 `cond` 的默认测试表达式。

    ```el
    ;; 好
    (cond
      ((< n 0) "negative")
      ((> n 0) "positive")
      (t "zero"))

    ;; 差
    (cond
      ((< n 0) "negative")
      ((> n 0) "positive")
      (:else "zero"))
    ```

* 用 `(1+ x)` & `(1- x)` 而不是 `(+ x 1)` 和 `(- x 1)`。

## 命名

> 程式设计的真正难题是替事物命名及使缓存失效。<br>
> ——Phil Karlton

* 方法与变量使用 `lisp-case`。

    ```el
    ;; 好
    (defvar some-var ...)
    (defun some-fun ...)

    ;; 差
    (defvar someVar ...)
    (defun somefun ...)
    (defvar some_fun ...)
    ```

* 用函数库的前缀避免命名冲突。

    ```el
    ;; 好
    (defun projectile-project-root ...)

    ;; 差
    (defun project-root ...)
    ```

* 给未使用到的局部（lexically scoped）变量名加前缀 `_`。

    ```el
    ;; 好
    (lambda (x _y) x)

    ;; 差
    (lambda (x y) x)
    ```

* 用 `--` 表示私有定义（比如：`projectile--private-fun`）。

* 断言函数（返回布尔值的函数）应该以 `p` 结尾，单字用 `p`、多字用 `-p`（比如：`evenp` 和 `buffer-live-p`）。

    ```el
    ;; 好
    (defun palindromep ...)
    (defun only-one-p ...)

    ;; 差
    (defun palindrome? ...) ; Scheme 风格
    (defun is-palindrome ...) ; Java 风格
    ```

## Macros

* 如果用函数能做到，不用宏。

* 先写个宏的用例，而后实现这个宏。

* 尽量把复杂的宏分解成较小的函数。

* 一个宏通常仅提供语法糖，其关键部分应该是函数。如此一来，能易与组合。

* 优先使用引号语法，而不是手动构造 list。

## Functions

* 局部绑定使用 `lambda`，hook 或全局变量不该使用 `lambda`。后者应该用有名称的函数，可读性更好、易于定制。


    ```el
    ;;; 好
    (mapcar (lambda (x) (or (car x) "")) some-list)
    (let ((predicate (lambda (x) (and (numberp x) (evenp x)))))
      (funcall predicate 1000))

    ;;; 差 - Define real functions for these.
    (defcustom my-predicate (lambda (x) (and (numberp x) (evenp x)))
      ...)
    (define-key my-keymap (kbd "C-f")
      (lambda () (interactive) (forward-char 1)))
    (add-hook 'my-hook (lambda () (save-some-buffers)))
    ```
* 永远不要 hard quote lambda，会导致无法 byte-compilation。

    ```el
    ;;; 好
    (lambda (x) (car x))

    ;;; 可以，但是不必要。
    #'(lambda (x) (car x))

    ;;; 差
    '(lambda (x) (car x))
    ```
* 不要用匿名函数 wrap 你不需要的函数。

    ```el
    ;; 好
    (cl-remove-if-not #'evenp numbers)

    ;; 差
    (cl-remove-if-not (lambda (x) (evenp x)) numbers)
    ```

* 用一个井号连着引号（`#'`）来 quote 函数名。它能给 byte-compiler 更好的提示：如果函数未定义，会给出警告。有些宏甚至会有不同效果（像 `cl-labels`）。

    ```el
    ;; 好
    (cl-remove-if-not #'evenp numbers)
    (global-set-key (kbd "C-l C-l") #'redraw-display)
    (cl-labels ((butterfly () (message "42")))
      (funcall #'butterfly))

    ;; 差
    (cl-remove-if-not 'evenp numbers)
    (global-set-key (kbd "C-l C-l") 'redraw-display)
    (cl-labels ((butterfly () (message "42")))
      (funcall 'butterfly))
    ```

### Macro Declarations

* 声明 [debug-specification](http://www.gnu.org/software/emacs/manual/html_node/elisp/Specification-List.html#Specification-List)，用来告诉 edebug 哪些参数会被执行。如果所有的参数都会被执行，用 `(declare (debug t))` 就够了。

* 声明 [indent specification](https://www.gnu.org/software/emacs/manual/html_node/elisp/Indenting-Macros.html#Indenting-Macros)，如果，macro 的参数不应该像函数（想想 `defun` 和 `with-current-buffer`）。

    ```el
    (defmacro define-widget (name &rest forms)
      "Description"
      (declare (debug (sexp body))
               (indent defun))
      ...)
    ```

### Loading and Autoloading

* 每个库函数文件的结果都需要一个 `provide` 语句和依据适当的注释（`provide` 允许其它库用 `require`  导入）。

    ```el
    (provide 'foo)

    ;;; foo.el ends here
    ```

* 用 `require` 加载库依赖，而不是 `load` 和 `load-library`（前者重复执行恒等，而后两者会重复执行）。

* 给 mode 定义和用户函数和命令（也就是，设置函数和应该绑定快捷键的命令）添加 autoload。相反的，不要给全局变量和内部函数添加 autoload。

    ```el
    ;;; 好
    ;;;###autoload
    (define-derived-mode foo-mode ...)

    ;;;###autoload
    (define-minor-mode foo-minor-mode ...)

    ;;;###autoload
    (defun foo-setup () ...)

    ;;; 差
    ;;;###autoload
    (defun foo--internal () ...)

    ;;;###autoload
    (defvar foo-option)
    ```

* 不要给非定义用的 top-level from（autoloading 一个永远不应该修改用户的配置功能）。但有一个例外：`auto-mode-alist` 可以用来改变新的 major mode。

    ```el
    ;;; 好
    ;;;###autoload
    (add-to-list 'auto-mode-alist '("\\.foo\\'" . foo-mode))

    ;;; 差
    ;;;###autoload
    (foo-setup)
    ```

## 注释

> 良好的代码是最佳的文档。当你要加一个注释时，扪心自问，“如何改善代码让它不需要注释？” 改善代码，再写相应文档使之更清楚。<br>
> ——Steve McConnell

* 尽量使你的代码自我文档化。

* 写标题的注释至少使用三个分号。

* 如果顶级注释表示一个标题，用三个分号，否则用两个分号。

* 在代码片段之前写注释，要与之对齐，使用两个分号。

* 写边缘注释使用一个分号。

* 分号和注释文字之间至少留一个空格。

    ```el
    ;;; Frob Grovel
    ;; This is where Frob grovels and where Grovel frobs.

    ;; This section of code has some important implications:
    ;;   1. Foo.
    ;;   2. Bar.
    ;;   3. Baz.

    (defun fnord (zarquon)
      ;; If zob, then veeblefitz.
      (quux zot
            mumble             ; Zibblefrotz.
            frotz))
    ```

* 注释超过一个单词了，应句首大写并使用标点符号。句号后使用两个空格。

* 避免肤浅的注释。

    ```el
    ;; 差
    (1+ counter) ; 计数器加一
    ```

* 及时更新注释。过时的注释比没有注释还要糟糕。

> 好代码就像是好的笑话 - 它不需要解释 <br>
> ——Russ Olsen

* 避免替烂代码写注释。重构代码让它们看起来一目了然。（要嘛就做，要嘛不做——不要只是试试看。——Yoda）

### 注解

* 注解应该直接写在相关代码那行之前。

* 注解关键字后面，跟着一个冒号及空格，接着是描述问题的文字。

* 如果需要用多行来描述问题，后续行要与第一行对齐。

* 用字母缩写和日期给你的注解打标签，使得它们的关联更易于辨识。

    ```el
    (defun some-fun ()
      ;; FIXME: This has crashed occasionally since v1.2.3. It may
      ;;        be related to the BarBazUtil upgrade. (xz 13-1-31)
      (baz))
    ```
* 在问题是显而易见的情况下，任何的文档会是多余的，注解应放在有问题的那行的最后，并且不需更多说明。这个用法应该是例外而不是规则。

    ```el
    (defun bar ()
      (sleep 100)) ; OPTIMIZE
    ```

* 使用 `TODO` 标记以后应加入的特征与功能。

* 使用 `FIXME` 标记需要修复的代码。

* 使用 `OPTIMIZE` 标记可能影响性能的缓慢或效率低下的代码。

* 使用 `HACK` 标记代码异味，即那些应该被重构的可疑编码习惯。

* 使用 `REVIEW` 标记需要确认其编码意图是否正确的代码。举例来说：`REVIEW: 我们确定用户现在是这么做的吗？`

* 如果你觉得恰当的话，可以使用其他定制的注解关键字，但别忘记录在项目的 `README` 或类似文档中。

### Docstrings

TODO

## Existential

* 保持一致性。在理想的世界里，遵循这些准则。
* 使用常识。

# 贡献

在本指南所写的每条规则都不是定案。这只是我渴望想与同样对 Emacs Lisp 编程风格有兴趣的大家一起工作，以致于最终我们可以替整个 Emacs 社区创造一个有益的资源。

欢迎 open tickets 或 push 一个带有改进的更新请求。在此提前感谢你的帮助！

# 授权

![Creative Commons License](http://i.creativecommons.org/l/by/3.0/88x31.png)
This work is licensed under a [Creative Commons Attribution 3.0 Unported License](http://creativecommons.org/licenses/by/3.0/deed.zh)

# 口耳相传

一份社区驱动的风格指南，如果没多少人知道，对一个社区来说就没有多少用处。微博转发这份指南，分享给你的朋友或同事。我们得到的每个评价、建议或意见都可以让这份指南变得更好一点。而我们想要拥有的是最好的指南，不是吗？

共勉之，<br>
[Bozhidar](https://twitter.com/bbatsov)

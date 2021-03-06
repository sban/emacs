# Installing Emacs with R on MacOS
## Installation of software
Install the package manager *homebrew*. Type the following code in the your terminal application (e.g. Terminal.app): 

```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Homebrew stores all programs in a folder named  'Celler' typically located at /usr/local/Cellar. The path is important as we later need to tell emacs where R is located.

3. Install *emacs* with homebrew. Type:
```
$ brew update
$ brew install emacs --with-cocoa
$ alias emacs="/usr/local/Cellar/emacs/26.1_1/Emacs.app/Contents/MacOS/Emacs -nw"
```

Alternative, you can download it here https://emacsformacosx.com/ (with no extras).



4. Open your version of GNU Emacs and locate the .emacs-file 
Open your Emacs application and type "c-x-c-f" to open a file and then type ".emacs". If the file does not allready exist, Emacs will create it.

Insert the following text to have a more smooth interface:

```
;;-------------------------------------------------------------------------------------------------
;; Faster startup
;;-------------------------------------------------------------------------------------------------
(setq gc-cons-threshold 64000000)
(add-hook 'after-init-hook #'(lambda ()
                               ;; restore after startup
                               (setq gc-cons-threshold 800000)))

;;-------------------------------------------------------------------------------------------------
;; ++     Personal info
;;-------------------------------------------------------------------------------------------------
(setq user-full-name "Mr XXX XXX"
      user-mail-address "XXX@XXX.XX")

;;-------------------------------------------------------------------------------------------------
;; ++     Settings
;;-------------------------------------------------------------------------------------------------
;; Set my default folder
(setq default-directory "~/Dropbox/" )

;; Set til UTF-8
(prefer-coding-system 'utf-8)
(when (display-graphic-p)
  (setq x-select-request-type '(UTF8_STRING COMPOUND_TEXT TEXT STRING)))

;; line wrap (text mode only)
 (add-hook 'text-mode-hook 'turn-on-visual-line-mode)
 (global-visual-line-mode 1)
 (setq org-startup-indented t) 

;; global line number 
(global-linum-mode t)
;; delete seleted text when typing
(delete-selection-mode t)  

;; remove beeps
(setq visible-bell nil
      ring-bell-function 'flash-mode-line)
(defun flash-mode-line ()
  (invert-face 'mode-line)
  (run-with-timer 0.1 nil #'invert-face 'mode-line))

;; WINDOWS MANAGEMENT
(global-set-key (kbd "<C-M-up>") 'shrink-window)
(global-set-key (kbd "<C-M-down>") 'enlarge-window)
(global-set-key (kbd "<C-M-left>") 'shrink-window-horizontally)
(global-set-key (kbd "<C-M-right>") 'enlarge-window-horizontally)

;; THEMES
 (load-theme 'misterioso)
 (menu-bar-mode -1)
 (tool-bar-mode -1)
 (scroll-bar-mode -1)
 (column-number-mode)
 (show-paren-mode)
 (global-hl-line-mode)
 (winner-mode t)

 ;; Start "blank"
  (setf inhibit-splash-screen t)
  (switch-to-buffer (get-buffer-create "new"))
  (delete-other-windows)

;; Change "yes or no" to "y or n"
 (fset 'yes-or-no-p 'y-or-n-p)

;; Autosave (off)
;disable backup
 (setq backup-inhibited t)
;disable auto save
 (setq auto-save-default nil)

;; Modeline customization
(setq-default mode-line-format
   (list
    ;; Current buffer name
    "%b "
    ;; Major mode
    "[%m] "
    ;; Modified status
    "[%*] "
    ;; Cursor position
    "Line: %l/%i"))
;; for "file: %f "

; Or just hide mode-line
;(setq-default mode-line-format nil)

;; CTR+z is undo
(global-set-key (kbd "C-z ") 'undo)

;; Zoom
(global-set-key (kbd "C-+") 'text-scale-increase)
(global-set-key (kbd "C--") 'text-scale-decrease)

;; Marking text
(delete-selection-mode t)
(transient-mark-mode t)
(setq x-select-enable-clipboard t)

;; Removing trailing spaces + remap for Mac-users
(global-set-key (kbd "M-S-SPC ") 'just-one-space)
(add-hook 'before-save-hook 'delete-trailing-whitespace)

;; REMOVE SCRATCH AND MESSAGES
;; Makes *scratch* empty.
(setq initial-scratch-message "")

;; Removes *scratch* from buffer after the mode has been set.
(defun remove-scratch-buffer ()
  (if (get-buffer "*scratch*")
      (kill-buffer "*scratch*")))
(add-hook 'after-change-major-mode-hook 'remove-scratch-buffer)

;; Removes *messages* from the buffer.
(setq-default message-log-max nil)
(kill-buffer "*Messages*")

;; Removes *Completions* from buffer after you've opened a file.
(add-hook 'minibuffer-exit-hook
      '(lambda ()
         (let ((buffer "*Completions*"))
           (and (get-buffer buffer)
                (kill-buffer buffer)))))

;; Don't show *Buffer list* when opening multiple files at the same time.
(setq inhibit-startup-buffer-menu t)

;; Show only one active window when opening multiple files at the same time.
(add-hook 'window-setup-hook 'delete-other-windows)

;;---------------------------------------------------------------------------
;; ++     PANDOC
;;---------------------------------------------------------------------------
;; Export to MS Word/docx formats
(add-hook 'markdown-mode-hook 'pandoc-mode)
(setq pandoc-binary "/usr/local/Cellar/pandoc/2.7.3/bin/pandoc")

;;---------------------------------------------------------------------------
;; ++ Dired
;;---------------------------------------------------------------------------
; less information about files
(add-hook 'dired-mode-hook 'dired-hide-details-mode)

;;---------------------------------------------------------------------------
;; ++     Movement-Stuff
;;---------------------------------------------------------------------------

;; Move between punctuation and comma
(defun xah-forward ()
  (interactive)
  (re-search-forward "\\.+\\|,+\\|;+" nil t))

(defun xah-backward ()
  (interactive)
  (re-search-backward "\\.+\\|,+\\|;+" nil t))

(global-set-key (kbd "C-æ") 'xah-backward)
(global-set-key (kbd "C-ø") 'xah-forward)

;; Move between paragraphs
(defun xah-forward-block (&optional n)
  "Move cursor beginning of next text block.
A text block is separated by blank lines.
This command similar to `forward-paragraph', but this command's behavior is the same regardless of syntax table.
URL `http://ergoemacs.org/emacs/emacs_move_by_paragraph.html'
Version 2016-06-15"
  (interactive "p")
  (let ((n (if (null n) 1 n)))
    (re-search-forward "\n[\t\n ]*\n+" nil "NOERROR" n)))

(defun xah-backward-block (&optional n)
  "Move cursor to previous text block.
See: `xah-forward-block'
URL `http://ergoemacs.org/emacs/emacs_move_by_paragraph.html'
Version 2016-06-15"
  (interactive "p")
  (let ((n (if (null n) 1 n))
        ($i 1))
    (while (<= $i n)
      (if (re-search-backward "\n[\t\n ]*\n+" nil "NOERROR")
          (progn (skip-chars-backward "\n\t "))
        (progn (goto-char (point-min))
               (setq $i n)))
      (setq $i (1+ $i)))))

(global-set-key (kbd "M-æ") 'xah-backward-block)
(global-set-key (kbd "M-ø") 'xah-forward-block)

;; Avy for jumping to visible text using a char-based decision tree
(global-set-key (kbd "C-:") 'avy-goto-char)
(global-set-key (kbd "M-g w") 'avy-goto-word-1)

;;-------------------------------------------------------------------------------------------------
;; ++     Repro's
;;-------------------------------------------------------------------------------------------------
(when (>= emacs-major-version 24)
  (require 'package)
  (package-initialize)
  (add-to-list 'package-archives '("melpa.milk" . "http://melpa.org/packages/") t) )


;;---------------------------------------------------------------------------
;; ++     Keybord-Mac-Stuff
;;---------------------------------------------------------------------------
(defun insert-backs ()  
    "insert back-slash"
    (interactive)
    (insert "\\ "))

;; Right option is ALT and left is META
;(setq mac-option-key-is-meta t)
;(setq mac-right-option-modifier nil)
;(global-set-key (kbd "M-:") 'insert-backs)

;; M is set to CMD (much easier)
(setq mac-option-modifier nil
mac-command-modifier 'meta
x-select-enable-clipboard t)

;; CTR+z is undo
(global-set-key (kbd "C-z ") 'undo)
```

Type "C-x-C-s" to save the file and then re-load your .emacs file by typing "M-x load-file"

# Setting up R (ess-mode)
We begin by installing R.

We do this by using Homebrew where we also install the Quartz-program (an application needed for the gui).

```
$ brew tap homebrew/science
$ install Caskroom/cask/quartz
$ brew install r
```

If the program is not installed, we also need XCode Command Line Tools (CLT), which is installed by the following command

```
$ xcode-select --install
```

To run R inside Emacs we need to install ESS. To do this, we need to download and install the program, which is done inside Emacs by the following procedure: "M-x "package-install" and then type "ess" 


Include the following code in your .emacs file

```
;;----------------------------------------------------------
;; ++     ESS: R
;;----------------------------------------------------------
(setq ess-ask-for-ess-directory nil)
(setq ess-local-process-name "R")
(setq-default inferior-R-program-name "/usr/local/Cellar/r/3.5.2_2/bin/R")
(require 'ess-site)
(setq ess-eval-visibly 'nowait)

;;;; create a new frame for each help instance
(setq ess-help-own-frame t)

;; Clear console
(defun clear-shell ()
   (interactive)
   (let ((old-max comint-buffer-maximum-size))
     (setq comint-buffer-maximum-size 0)
     (comint-truncate-buffer)
     (setq comint-buffer-maximum-size old-max))) 
 (global-set-key  (kbd "\C-x y") 'clear-shell)
 
;; Read pdfs outside emacs
(require 'openwith)
(openwith-mode t)
(setq openwith-associations '(("\\.pdf\\'" "open" (file))))
 
 ;; Autocomplete
(require 'auto-complete-config)
(ac-config-default)
(setq ess-use-auto-complete t)
 
;; Make the arrow keys refer to previous commands
(defun my-ess-mode-hook ()
(local-set-key '[up] 'comint-previous-input)
(local-set-key '[down] 'comint-next-input))

(add-hook 'inferior-ess-mode-hook 'my-ess-mode-hook) 
 
```
R will start but produces some errors. To avioid these modify your .Renviron-file:

```
emacs ~/.Renviron
```

An change the text to this:

```
LANGUAGE=dk

LC_ALL = "en_US.UTF-8"
```

Finally to get rJava to work, we need to create an R profile 

```
emacs ~/.Profile
```

that include the following code:

```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-10.0.2.jdk/Contents/Home
```

Please remeber to change "jdk-10.0.2.jdk" with the version that you have installed on your computer.

# Setting up LaTeX
First we need to use Homebrew to install the Tex-compiler:

```
$ brew cask install mactex
$ brew cask install texmaker
```

Tex Live Utility that enables easy management of the LaTex-packages. This program needs to be updated after the installation. Use spotlight to find and open the program and follow the instructions.

Find and install "auctex" using "M-x list-packages".
Include the following code in your .emacs file:

```
;;-------------------------------------------------------------------------------------------------
;; ++     LaTeX
;;-------------------------------------------------------------------------------------------------
(setq TeX-auto-save t)
(setq TeX-parse-self t)
(setq TeX-save-query nil)
;(setq TeX-PDF-mode t)

(setenv "PATH" (concat (getenv "PATH") ":/Library/Tex/texbin/"))
(setq exec-path (append exec-path '(/Library/Tex/texbin/"))

(setq org-latex-pdf-process '("latexmk -pdflatex='%latex -shell-escape -interaction nonstopmode' -pdf -output-directory=%o -f %f"))

;; Automatic latex-template when creating a .tex file
(auto-insert-mode)
(setq auto-insert-directory "/Users/stefan/Dropbox/MISC/emacs/templates/")
(setq auto-insert-query nil)
(define-auto-insert "\\.tex$" "my-latex-template.tex")

;;  AUCTeX has an auto-complete function
(add-hook 'LaTeX-mode-hook
      (lambda()
        (local-set-key [C-tab] 'TeX-complete-symbol)))

```

# Setting up org-mode
In the recent versions of emacs, org-mode is installed by default. However, to gain acces to the nice reference-manager you need to install org-ref.

First install the org-ref package using the m-x list-package. 

Then include the following code in your .emacs file:

```
(require 'org-ref)
(require 'org-ref-pdf)
(require 'org-ref-url-utils)
(require 'org-ref-latex)
(setq reftex-default-bibliography '("/Users/XXX/Dropbox/MISC/emacs/bib/ref.bib"))

(setq org-ref-bibliography-notes "/Users/XXX/Dropbox/MISC/emacs/notes.org"
      org-ref-default-bibliography '("/Users/XXX/Dropbox/MISC/emacs/bib/ref.bib")
      org-ref-pdf-directory "/Users/XXX/Dropbox/MISC/emacs/bibtex-pdfs/")


(setq bibtex-completion-bibliography "/Users/XXX/Dropbox/MISC/emacs/bib/ref.bib"
      bibtex-completion-library-path "/Users/XXX/Dropbox/MISC/emacs/bibtex-pdfs"
      bibtex-completion-notes-path "/Users/XXX/Dropbox/MISC/emacs/notes.org")

(unless (file-exists-p org-ref-pdf-directory)
 (make-directory org-ref-pdf-directory t))

(setq org-src-fontify-natively t
     org-confirm-babel-evaluate nil
     org-src-preserve-indentation t)

(setq org-latex-pdf-process (list "latexmk -shell-escape -bibtex -f -pdf %f"))
```

Remember that for the reference to show up when exporting a file you nedd to include the following code at the start of the document:

```
#+LATEX_HEADER: \usepackage{natbib}
```

And this code at the end of the document:

```
bibliographystyle:plain
bibliography:/Users/XXX/Dropbox/MISC/emacs/bib/ref.bib
```

Finally, to install a spelling program, use Homebrew to install ispell

```
$ brew install ispell
```

And add the following code to your .emacs file

```
;;---------------------------------------------------------------------------
;; ++     SPELLING
;;---------------------------------------------------------------------------
(setq ispell-program-name "/path/to/ispell")

(dolist (hook '(text-mode-hook))
  (add-hook hook (lambda () (flyspell-mode 1))))

(global-set-key (kbd "<f8>") 'ispell-word)
(defun flyspell-check-next-highlighted-word ()
  "Custom function to spell check next highlighted word"
  (interactive)
  (flyspell-goto-next-error)
  (ispell-word))
(global-set-key (kbd "M-<f8>") 'flyspell-check-next-highlighted-word)
```

# Python
To enable python programming install elpy (using m-x list-programs).

To enable natural language processing (NLP) install nltk using pip in the terminal
```
 $ pip3 install nltk
```


Insert the following code into your .emacs file:

```
;;---------------------------------------------------------------------------
;; ++    PYTHON
;;---------------------------------------------------------------------------
(elpy-enable)
(setq elpy-rpc-python-command "python3")
(setq python-shell-interpreter "/usr/local/bin/python3")
(setq org-babel-python-command "/usr/local/bin/python3")
```

Other addition features:

;;---------------------------------------------------------------------------
;; ++    ORG-BABEL
;;---------------------------------------------------------------------------
(org-babel-do-load-languages
 'org-babel-load-languages
 '(
   (emacs-lisp . t)
   (org . t)
   (shell . t)
   (python . t)
   (gnuplot . t)
   (R . t)
   (latex . t)
   (dot . t)
   (awk . t)
   ))

(org-download-enable)

(setq org-startup-align-all-tables t)
(org-table-map-tables 'org-table-iterate)

;;---------------------------------------------------------------------------
;; ++   Writer-Mode
;;---------------------------------------------------------------------------
(defun writing-mode ()
  (interactive)
  (setq buffer-face-mode-face '(:family "Monospace" :height 250))
  (buffer-face-mode)
  (linum-mode 0)
  (writeroom-mode 1)
  (blink-cursor-mode)
  (visual-line-mode 1)
  (setq truncate-lines nil)
  (setq-default line-spacing 10)
 (setq global-hl-line-mode nil)
 )

(with-eval-after-load 'writeroom-mode
  (define-key writeroom-mode-map (kbd "C-M-æ") #'writeroom-decrease-width)
  (define-key writeroom-mode-map (kbd "C-M-ø") #'writeroom-increase-width)
  (define-key writeroom-mode-map (kbd "C-M-=") #'writeroom-adjust-width))

;; Calender
(require 'calfw)
(require 'calfw-org)

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
;; ++     Personal info
;;-------------------------------------------------------------------------------------------------

(setq user-full-name "Mr XXX XXX"
      user-mail-address "XXX@XXX.XX")

;;-------------------------------------------------------------------------------------------------
;; ++     Settings
;;-------------------------------------------------------------------------------------------------
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

;; remove beeps and flash mode-line instead
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

;; Hide mode-line
(setq-default mode-line-format nil)

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

(global-set-key (kbd "M-p") 'xah-backward-block)
(global-set-key (kbd "M-n") 'xah-forward-block)

;;-------------------------------------------------------------------------------------------------
;; ++     Repro's
;;-------------------------------------------------------------------------------------------------
(setq package-archives (append package-archives
       '(("melpa" . "http://melpa.org/packages/")
       ("marmalade" . "http://marmalade-repo.org/packages/")
       ("gnu" . "http://elpa.gnu.org/packages/")
       ;; ("org" . "http://orgmode.org/elpa/")
       ("elpy" . "http://jorgenschaefer.github.io/packages/"))))
(package-initialize)

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
;;-------------------------------------------------------------------------------------------------
;; ++     R
;;-------------------------------------------------------------------------------------------------
(setq ess-ask-for-ess-directory nil) 
(setq ess-local-process-name "R") 
(setq-default inferior-R-program-name "/usr/local/Cellar/r/3.5.1/bin/R")

;; Clear console
(defun clear-shell ()
   (interactive)
   (let ((old-max comint-buffer-maximum-size))
     (setq comint-buffer-maximum-size 0)
     (comint-truncate-buffer)
     (setq comint-buffer-maximum-size old-max))) 
 (global-set-key  (kbd "\C-x c") 'clear-shell)
 
 ;; Autocomplete
(require 'auto-complete-config)
(ac-config-default)
(setq ess-use-auto-complete t)
 
```

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

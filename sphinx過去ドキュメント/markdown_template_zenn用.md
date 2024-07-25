# title
Sphinx備忘録
# 環境
```bash
Python 3.8.10
$ pip freeze | grep sphinx
sphinx-rtd-theme==1.1.1
sphinxcontrib-devhelp==1.0.2
sphinxcontrib-htmlhelp==2.0.0
sphinxcontrib-jsmath==1.0.1
sphinxcontrib-qthelp==1.0.3
sphinxcontrib-serializinghtml==1.1.5
sphinxcontrib.applehelp==1.0.3
(FACE01) 
$ inxi -SCGxx --filter
System:    Kernel: 5.15.0-46-generic x86_64 bits: 64 compiler: N/A Desktop: Unity wm: gnome-shell dm: GDM3 
           Distro: Ubuntu 20.04.4 LTS (Focal Fossa) 
CPU:       Topology: Quad Core model: AMD Ryzen 5 1400 bits: 64 type: MT MCP arch: Zen rev: 1 L2 cache: 2048 KiB 
Graphics:  Device-1: NVIDIA TU116 [GeForce GTX 1660 Ti] vendor: Micro-Star MSI driver: nvidia v: 515.65.01 bus ID: 08:00.0 
```

# これはなにか
自動ドキュメント生成のためにSphinxを使っています。アプリケーションのバージョンアップのたびに自動化スクリプトによってSphinxを走らせるのですが、うまくドキュメントが生成されない事を何度か経験しました。事前知識が曖昧なことで不具合解消に時間を取られるので、一度きちんと整理したいと思い備忘録的に書いていきたいと思います。

# 発生する問題
ルートディレクトリに存在する自動化スクリプト(version_control.py)が実行されてしまう。
## 予想する大まかな対処法
- sphinx-apidocの`EXCLUDE_PATTERN`
- conf.py中の`sys.path.insert(0, os.path.abspath())`や`sys.path.append`
- conf.py中の`exclude_patterns = []`

### 対処
- sphinx-apidoc
    `sphinx-apidoc -f -o ./sphinx '/home/terms/bin/FACE01' version_control.py system_check.py compile.py`
- `sphinx/conf.py`
```python
import os
import sys
sys.path.insert(0, os.path.abspath(".."))
```

```python
# 有効に動作しない
exclude_patterns = [
    '_build',
    'Thumbs.db',
    '.DS_Store',
    'system_check.py',
    'compile.py',
    'CTKtest.py',
    'version_control.py'
]
```

# 基本的なsphinxでのドキュメント作成手順
```bash
# pip install
pip install sphinx sphinx_rtd_theme

# working dir にsphinxフォルダを作成
if [ ! -d ./sphinx ]; then mkdir ./sphinx; fi
if [ ! -d ./docs ]; then mkdir ./docs; fi

# docsフォルダに`.nojekyll`ファイルがないとgithub pages上で表示が崩れる原因になる
if [ -f ./docs/.nojekyll ]; then
    touch ./docs/.nojekyll

# sphinx_bkフォルダからconf.pyとindex.rst他を複製
cp -f -r sphinx_bk/origin/* sphinx/

# sphinx-apidocに除外ファイルパターンを追加
sphinx-apidoc -f -o /home/terms/bin/FACE01/sphinx /home/terms/bin/FACE01 version_control.py system_check.py compile

# 上書きbuild
sphinx-build -b html -E ./sphinx ./docs
```

# 自動化スクリプトの該当部分
```python
"""②SphinxによるDocファイルの自動生成
sphinxエラーまとめ.mdファイルを参照
"""
print("②SphinxによるDocファイルの自動生成")
subprocess.run(["play -v 0.5 -q voices/005スフィンクスを起動.wav"], shell=True, check=True)

# sphinx_bkフォルダからconf.pyとindex.rst他を複製
try:
    sphinx_apidoc: list = ["cp -f -r sphinx_bk/origin/* sphinx/"]
    ret = subprocess.run(sphinx_apidoc, shell=True, check=True, cwd="/home/terms/bin/FACE01")
except subprocess.CalledProcessError as e:
    print(e)
    exit(1)

try:
    sphinx_apidoc: list = ["sphinx-apidoc -f -o /home/terms/bin/FACE01/sphinx /home/terms/bin/FACE01 version_control.py system_check.py compile"]
    # sphinx_apidoc: list = ["sphinx-apidoc", "-f", "-R", ver, "-o /home/terms/bin/FACE01/sphinx /home/terms/bin/FACE01"]
    ret = subprocess.run(sphinx_apidoc, shell=True, check=True, cwd="/home/terms/bin/FACE01")
except subprocess.CalledProcessError as e:
    print(e)
    exit(1)

try:
    sphinx_cmd:list = ["sphinx-build -b html -E /home/terms/bin/FACE01/sphinx /home/terms/bin/FACE01/docs"]
    ret = subprocess.run(sphinx_cmd, shell=True, check=True)
except subprocess.CalledProcessError as e:
    print(e)
    exit(1)
```
理由は分かりませんが`sphinx_apidoc: list = ["sphinx-apidoc", "-f", "-R", ver, "-o /home/terms/bin/FACE01/sphinx /home/terms/bin/FACE01"]`とするとエラーが発生します。これについては別途調べる予定です。

# sphinx/conf.py
```python
# Configuration file for the Sphinx documentation builder.
"""自動ドキュメント生成のためのconf.py.

files:
    ~/bin/FACE01/sphinx_bk/conf.py
    ~/bin/FACE01/sphinx_bk/index.rst
"""
# For the full list of built-in configuration values, see the documentation:
# https://www.sphinx-doc.org/en/master/usage/configuration.html

import os
import sys


# [ os.path.abspath(path)](https://docs.python.org/ja/3/library/os.path.html#os.path.abspath)
# os.path.abspath: (function)
# abspath(path: PathLike[AnyStr@abspath]) -> AnyStr@abspath
# abspath(path: AnyStr@abspath) -> AnyStr@abspath
# Return an absolute path.
# sys.path.insert: (method) insert(__index: SupportsIndex, __object: str, /) -> None
sys.path.insert(0, os.path.abspath(".."))
# sys.path.append: (method) append(__object: str, /) -> None
sys.path.append(os.path.abspath("../example"))
sys.path.append(os.path.abspath("../face01lib"))
sys.path.append(os.path.abspath("../face01lib/models"))

# -- Project information -----------------------------------------------------
# https://www.sphinx-doc.org/en/master/usage/configuration.html#project-information
project = 'FACE01'
copyright = '2023, yKesamaru'
author = 'yKesamaru'
release = '1.4.11'

# -- General configuration ---------------------------------------------------
# https://www.sphinx-doc.org/en/master/usage/configuration.html#general-configuration

extensions = [
    'sphinx.ext.napoleon',
    'sphinx.ext.autodoc',
]

# extensions = [
#     'sphinx.ext.napoleon',
#     'myst_parser',
#     'sphinx.ext.autodoc',
# ]

# [Markdown](https://www.sphinx-doc.org/en/master/usage/markdown.html)
source_suffix = {
    '.rst': 'restructuredtext'
}

# source_suffix = {
#     '.rst': 'restructuredtext',
#     '.md': 'markdown',
# }

# exclude_patterns = [
#     '_build',
#     'Thumbs.db',
#     '.DS_Store'
# ]

exclude_patterns = [
    '_build',
    'Thumbs.db',
    '.DS_Store',
    'system_check.py',
    'compile.py',
    'CTKtest.py',
    'version_control.py'
]

autodoc_mock_imports = [
    "anti_spoof",
    "lightweight_GUI",
    "compile",
]

# [sphinx.ext.napoleon – Support for NumPy and Google style docstrings](https://www.sphinx-doc.org/en/master/usage/extensions/napoleon.html)
# Napoleon settings
# [Configuration](https://www.sphinx-doc.org/en/master/usage/extensions/napoleon.html#configuration)
napoleon_google_docstring = True
napoleon_numpy_docstring = False
napoleon_include_init_with_doc = False
napoleon_include_private_with_doc = False
napoleon_include_special_with_doc = True
napoleon_use_admonition_for_examples = True
napoleon_use_admonition_for_notes = False
napoleon_use_admonition_for_references = False
napoleon_use_ivar = False
napoleon_use_param = True
napoleon_use_keyword = True
napoleon_use_rtype = True
napoleon_preprocess_types = False
napoleon_type_aliases = None
napoleon_attr_annotations = True
napoleon_custom_sections = None


# -- Options for HTML output -------------------------------------------------
# https://www.sphinx-doc.org/en/master/usage/configuration.html#options-for-html-output
html_theme = 'sphinx_rtd_theme'  # Read the Docs
html_static_path = ['_static']
templates_path = ['_templates']
html_logo = 'https://raw.githubusercontent.com/yKesamaru/FACE01_SAMPLE/master/images/Logo_dist.png'
html_favicon = 'https://raw.githubusercontent.com/yKesamaru/FACE01_SAMPLE/master/images/Logo.ico'
# html_additional_pages = {
#     'Tokai kaoninsho': 'https://tokai-kaoninsho.com/',
#     'GitHub': 'https://github.com/yKesamaru/FACE01_SAMPLE'
# }
html_copy_source = False
html_show_sourcelink = False
```

# sphinx/index.rst
```python
.. FACE01 documentation master file, created by
   sphinx-quickstart on Thu Sep 29 11:28:46 2022.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

FACE01 Document
==================================

This document is a exhaustive man page for FACE01's public class, methods and some simple examples.

.. Caution::
   Please note that this is automated to always follow the latest version (v1.4.11), so not compatible with previous version of FACE01.

See bellow for a simpler way to use FACE01.

`Step-by-step to use FACE01 library <https://github.com/yKesamaru/FACE01_SAMPLE/blob/master/docs/example_doc.md>`_

..
   .. include:: ../docs/functions.md


.. toctree::
   :maxdepth: 4
   :caption: Contents:

   example
   face01lib
   models
   
   GitHub <https://github.com/yKesamaru/FACE01_SAMPLE>
   Tokai-kaoninsho <https://tokai-kaoninsho.com/>


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
```

# 結論

# 詳細

# 参考リンク

# あとがき

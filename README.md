# code-quality-talk - Notes:

## pre-commit demo:

- Create a new repo directory
- `cd dev`
- `mkdir pre-commit-demo`
- `cd pre-commit-demo`
- `git init`
- `pip3 install pre-commit --user`
- `pre-commit install`
- `pre-commit sample-config > .pre-commit-config.yaml`
- `pre-commit install-hooks`
- ```
  echo "Hello World    " > hello.txt
  ```
- `git add .`
- `pre-commit run --all-files`  (show that it fails)
- `git diff`                    (show that it has removed the space that we added)
- `pre-commit run --all-files`  (show that it passes second time)
- `truncate -s -1 hello.txt`    (remove the trailing space from the file)
- `pre-commit run --all-files`  (show that it fails on the end of file fixer)
- `pre-commit run --all-files`  (show that it passes second time)


## pre-commit demo on LincolnHack repo:

### Basics:

- `cd lincolnhack_dd`
- `git switch spike`
- `pip3 install pre-commit --user`
- `git switch -c pre-commit`
- `pre-commit install`
- ```
  cat << EoI > .pre-commit-config.yaml
  repos:
    - repo: https://github.com/pre-commit/pre-commit-hooks
      rev: v4.4.0
      hooks:
        - id: check-added-large-files
          args: [‘--maxkb=1024’]
        - id: check-executables-have-shebangs
        - id: check-json
        - id: check-merge-conflict
        - id: check-shebang-scripts-are-executable
        - id: check-symlinks
        - id: check-yaml
          args: ['--unsafe']
        - id: detect-private-key
        - id: end-of-file-fixer
        - id: mixed-line-ending
          args: [--fix=lf]
        - id: trailing-whitespace
        # Python
        - id: check-docstring-first
  EoI
  ```
- `pre-commit install-hooks`
- `git add .`
- `pre-commit run --all-files`

### Python specific:

- Add cspell, black and isort to the .pre-commit-config.yaml
  - ```
      - repo: https://github.com/pycqa/isort
        rev: 5.12.0
        hooks:
          - id: isort

      - repo: https://github.com/psf/black
        rev: 23.7.0
        hooks:
          - id: black
            args: ['--line-length', '120']
    ```
- Copy config for isort to `.isort.cfg`
  - ```
    [settings]
    line_length=120
    lines_after_imports = 2
    force_sort_within_sections=True
    import_heading_future=Futures Imports
    import_heading_stdlib=Standard Library Imports
    import_heading_django=Django Imports
    import_heading_thirdparty=3rd Party Imports
    import_heading_firstparty=App Imports
    import_heading_localfolder=Local Imports
    known_first_party = lincolnhack_dd
    known_django = django
    sections = FUTURE,STDLIB,DJANGO,THIRDPARTY,FIRSTPARTY,LOCALFOLDER
    skip=migrations
    ```
- Run `pre-commit run --all-files`
- Add in fixes
- Run `pre-commit run --all-files` again
- Add cspell to the .pre-commit-config.yaml
  - ```
      - repo: https://github.com/streetsidesoftware/cspell-cli
        rev: v6.31.0
        hooks:
          - id: cspell
    ```
- Run `pre-commit run --all-files`
- Add `cspell.config.yaml` file
  - ```
    ---
    allowCompoundWords: true
    languageSettings:
      - caseSensitive: false
        enabled: true
        languageId: en-GB
    dictionaryDefinitions:
      - name: my_words
        path: ./.my_words.txt
        addWords: true
    dictionaries:
      - my_words
    ```
- Add `.my_words.txt` file
  - Show how to run specific hooks
  - Add some words to `.my_words.txt`
  - Show that words are now not flagged
  - Show disabling single line
    - `# // cspell:disable-line`
  - Highlight spelling mistake and fix it
  - Add changed files


## Github Actions demo:

### Reusing the LincolnHack demo:

- Create .github/workflows folder
- Create .github/workflows/pre-commit.yml
  - ```
    name: pre-commit

    on: [push]

    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v3
          - name: Set up Python 3.11
            uses: actions/setup-python@v4
            with:
              python-version: '3.11'
              check-latest: true
              cache: 'pip'
              cache-dependency-path: .github/requirements.txt
          - run: python3 -m pip install -r .github/requirements.txt
          - run: pre-commit install --install-hooks
          - run: pre-commit run --show-diff-on-failure --color=never --all-files
    ```
- Create .github/requirements.txt
  - ```
    pre-commit==3.3.3
    ```
- Commit and push (Have Github Actions tab open and ready to show)

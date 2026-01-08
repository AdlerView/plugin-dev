# Known Text File Extensions

Comprehensive list of file extensions that indicate text-based content suitable for reading and description generation.

## Source Code

### Web Development
- `.html`, `.htm` - HTML markup
- `.css`, `.scss`, `.sass`, `.less` - Stylesheets
- `.js`, `.mjs`, `.cjs` - JavaScript
- `.ts`, `.mts`, `.cts` - TypeScript
- `.jsx`, `.tsx` - React components
- `.vue`, `.svelte` - Framework components

### Systems Programming
- `.c`, `.h` - C source and headers
- `.cpp`, `.cc`, `.cxx`, `.hpp`, `.hxx` - C++ source and headers
- `.rs` - Rust
- `.go` - Go
- `.zig` - Zig

### Mobile Development
- `.swift` - Swift
- `.kt`, `.kts` - Kotlin
- `.java` - Java
- `.m`, `.mm` - Objective-C

### Scripting
- `.py`, `.pyw`, `.pyi` - Python
- `.rb`, `.erb` - Ruby
- `.pl`, `.pm` - Perl
- `.php` - PHP
- `.lua` - Lua
- `.r`, `.R` - R

### Shell Scripts
- `.sh` - Bourne shell
- `.bash` - Bash
- `.zsh` - Zsh
- `.fish` - Fish
- `.ps1`, `.psm1` - PowerShell
- `.bat`, `.cmd` - Windows batch

### Functional Languages
- `.hs`, `.lhs` - Haskell
- `.ml`, `.mli` - OCaml
- `.ex`, `.exs` - Elixir
- `.erl`, `.hrl` - Erlang
- `.clj`, `.cljs`, `.cljc` - Clojure
- `.lisp`, `.cl`, `.el` - Lisp/Emacs Lisp
- `.scm`, `.ss` - Scheme

### Other Languages
- `.scala`, `.sc` - Scala
- `.groovy` - Groovy
- `.nim` - Nim
- `.cr` - Crystal
- `.d` - D
- `.v` - V
- `.asm`, `.s` - Assembly

## Configuration Files

### Data Formats
- `.json`, `.jsonc`, `.json5` - JSON
- `.yaml`, `.yml` - YAML
- `.toml` - TOML
- `.xml`, `.xsd`, `.xsl` - XML
- `.csv`, `.tsv` - Delimited data
- `.ini`, `.cfg`, `.conf` - INI-style config

### Environment & Secrets
- `.env`, `.env.example`, `.env.local` - Environment variables
- `.properties` - Java properties

### Package Managers
- `package.json` - npm/Node.js
- `Cargo.toml` - Rust
- `go.mod`, `go.sum` - Go
- `Gemfile`, `Gemfile.lock` - Ruby
- `requirements.txt`, `Pipfile`, `pyproject.toml` - Python
- `composer.json` - PHP
- `Package.swift` - Swift

### Build & CI
- `Makefile`, `makefile`, `GNUmakefile` - Make
- `CMakeLists.txt` - CMake
- `Dockerfile`, `docker-compose.yml` - Docker
- `.travis.yml` - Travis CI
- `.github/workflows/*.yml` - GitHub Actions
- `.gitlab-ci.yml` - GitLab CI
- `Jenkinsfile` - Jenkins

### Editor & IDE
- `.editorconfig` - EditorConfig
- `.prettierrc`, `.prettierrc.json` - Prettier
- `.eslintrc`, `.eslintrc.json` - ESLint
- `tsconfig.json`, `jsconfig.json` - TypeScript/JavaScript
- `.vscode/*.json` - VS Code settings

## Documentation

### Markdown & Text
- `.md`, `.markdown`, `.mdown` - Markdown
- `.txt`, `.text` - Plain text
- `.rst`, `.rest` - reStructuredText
- `.adoc`, `.asciidoc` - AsciiDoc
- `.org` - Org-mode

### API Documentation
- `.openapi.yaml`, `.openapi.json` - OpenAPI
- `.graphql`, `.gql` - GraphQL

### Special Files (case-insensitive)
- `README`, `README.*` - Project readme
- `LICENSE`, `LICENSE.*` - License files
- `CHANGELOG`, `CHANGELOG.*` - Change logs
- `CONTRIBUTING`, `CONTRIBUTING.*` - Contribution guidelines
- `AUTHORS`, `MAINTAINERS` - Author lists

## Git & Version Control

- `.gitignore` - Git ignore patterns
- `.gitattributes` - Git attributes
- `.gitmodules` - Git submodules
- `.gitconfig` - Git configuration
- `.hgignore` - Mercurial ignore

## Database

- `.sql` - SQL scripts
- `.prisma` - Prisma schema
- `.graphql` - GraphQL schemas

## Templates

- `.ejs` - EJS templates
- `.hbs`, `.handlebars` - Handlebars
- `.pug`, `.jade` - Pug/Jade
- `.mustache` - Mustache
- `.jinja`, `.jinja2`, `.j2` - Jinja

## Detection Fallback

For files without a recognized extension, use the `file` command to check MIME type:

```bash
file --mime-type -b "$filepath"
```

If the result starts with `text/`, the file is text-based and can be read.

Common text MIME types:
- `text/plain`
- `text/html`
- `text/css`
- `text/javascript`
- `text/x-python`
- `text/x-shellscript`
- `application/json`
- `application/xml`
- `application/x-yaml`

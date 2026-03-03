# AGENTS.md - Developer Guide for rime-ice

This is the configuration repository for [Rime Input Method](https://rime.im/). It contains:
- YAML schema files for input schemes (full pinyin, double pinyin, etc.)
- Chinese and English dictionaries
- Lua plugins for Rime functionality
- Go tools for dictionary management in `others/script/`

## Build, Test, and Run Commands

### Go Scripts (others/script/)

The Go code manages dictionary building and validation.

```bash
# Navigate to the Go module
cd others/script

# Install dependencies
go mod tidy

# Run the main tool (full workflow)
go run main.go --rime_path /path/to/rime-ice --auto_confirm

# Run specific subcommands:
# s - Sort and deduplicate dictionaries
go run main.go s

# t - Temporary/test function
go run main.go t

# tp - Pinyin annotation
go run main.go tp

# Run with custom rime path
go run main.go --rime_path "$GITHUB_WORKSPACE" --auto_confirm
```

### Running Tests

There are **no unit tests** in this repository. The Go code does not have `_test.go` files.

To validate dictionaries, use the `go run main.go` command which runs validation checks.

### Linting

No formal linting tools are configured. The Go code follows standard Go conventions.

### Build (CI)

The build is triggered by:
1. Commit message containing `[build]`
2. Manual workflow dispatch in GitHub Actions

The workflow (`.github/workflows/release.yml`):
- Runs `go mod tidy` and `go run main.go` 
- Creates distribution zip files for release

---

## Code Style Guidelines

### Go Code (others/script/)

Follow standard Go conventions:

- **Imports**: Grouped and sorted alphabetically
  ```go
  import (
      "bufio"
      "fmt"
      "log"
      "os"
      "path"
      "strconv"
      "strings"
      "sync"
      "time"
      "unicode"
      "unicode/utf8"

      mapset "github.com/deckarep/golang-set/v2"
  )
  ```

- **Naming**: 
  - Use camelCase for variables and functions
  - Use PascalCase for exported functions
  - Use snake_case for file names (e.g., `check.go`, `pinyin.go`)

- **Error Handling**: Use `log.Fatalln(err)` for fatal errors, return errors otherwise

- **Concurrency**: Use `sync.WaitGroup` for parallel processing

- **Code Comments**: Comments in Chinese (since this is a Chinese project)

### Lua Code (lua/)

Rime Lua plugins follow standard Lua patterns:

- **Module Structure**:
  ```lua
  local M = {}

  function M.init(env)
      -- initialization code
  end

  function M.func(input, seg, env)
      -- main logic
  end

  return M
  ```

- **Naming**: Use camelCase for functions/variables, snake_case for file names

- **Comments**: Use Chinese comments to describe functionality

- **Common Patterns**:
  - Use `yield(cand)` to return candidates
  - Use `Candidate(type, start, end, text, comment)` to create candidates
  - Access config via `env.engine.schema.config`

### YAML Configuration Files

- **Schema files** (`.schema.yaml`): Define input method behavior
- **Dictionary files** (`.dict.yaml`): Contain word lists with format: `word\tpinyin\tweight`
- **Format**: Tab-separated values, weights are integers
- **Comments**: Start with `#`

### Dictionary File Format

```
word1	pinyin1	100
word2	pinyin1 pinyin2	90
# 注释
```

### General Guidelines

1. **File Organization**:
   - Schema files in root directory
   - Dictionaries in `cn_dicts/`, `en_dicts/` directories
   - Lua plugins in `lua/` directory
   - Go tools in `others/script/` directory

2. **Dictionary Maintenance**:
   - Use the Go tool to validate and sort dictionaries
   - Follow the weight convention: higher = more frequent
   - Pinyin should be lowercase, space-separated for multi-syllable words

3. **Version Control**:
   - Commit message format: `type(scope): message` or just descriptive messages
   - Use `[build]` tag in commit to trigger dictionary rebuild

4. **Testing Changes**:
   - Copy files to Rime user directory
   - Redeploy Rime to apply changes
   - Test input method functionality

---

## Project Structure

```
.
├── *.schema.yaml        # Input scheme definitions
├── *.dict.yaml         # Dictionary files
├── cn_dicts/           # Chinese dictionaries
│   ├── 8105.dict.yaml  # Common characters
│   ├── 41448.dict.yaml # Large character set
│   ├── base.dict.yaml # Base vocabulary
│   ├── ext.dict.yaml  # Extended vocabulary
│   └── tencent.dict.yaml # Tencent vocabulary
├── en_dicts/           # English dictionaries
│   ├── en.dict.yaml
│   └── en_ext.dict.yaml
├── lua/                # Lua plugins for Rime
│   ├── date_translator.lua
│   ├── unicode.lua
│   └── ...
├── opencc/            # OpenCC configuration
└── others/script/    # Go tools for dictionary management
    ├── main.go
    └── rime/
        ├── check.go
        ├── pinyin.go
        └── ...
```

---

## Common Tasks

### Adding New Words

1. Add words to appropriate dictionary file in `cn_dicts/` or `en_dicts/`
2. Run `go run main.go` to validate and re-sort

### Creating New Input Scheme

1. Copy an existing `.schema.yaml` as template
2. Modify the schema configuration
3. Create corresponding `.dict.yaml` file

### Modifying Lua Plugins

1. Edit files in `lua/` directory
2. Redeploy Rime to test changes

### Running Dictionary Build Locally

```bash
cd others/script
go run main.go --rime_path /home/guozr/CODE/rime-ice --auto_confirm
```

---

## Notes

- This is primarily a configuration/data repository, not a typical software project
- The main development work involves maintaining dictionaries and YAML configurations
- The Go code is a utility for dictionary validation and processing
- No formal code review process; changes are committed directly

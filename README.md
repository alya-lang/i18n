# i18n

[![CI](https://github.com/alya-lang/i18n/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/i18n/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/i18n?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fi18n%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fi18n%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

Lightweight internationalization: locale bundles, gettext-style .tr and JSON loaders, CLDR-lite plurals

---

## 🌟 Features

- 🌍 **Locale Bundles**: `I18nBundle` tables with active language, fallback chain (lang → base → fallback), and explicit-language lookup
- 📄 **gettext-style `.tr` Loader**: Key/value entries with `-----` separators, multiline values, CRLF-safe
- 🗂️ **Flat JSON Loader**: String translation files with literal dotted keys, language subdirectories as namespaces (`de/app.json` → `app.*`)
- 📊 **CSV/TSV Loader**: `key,value` records per `<lang>.csv` / `<lang>.tsv` file (RFC-4180 quoting)
- 📝 **YAML Loader**: Flat string mappings per `<lang>.yaml` / `<lang>.yml`
- 📦 **TOML Loader**: `key = "value"` pairs per `<lang>.toml`, `[table]` sections map to dotted keys (`[menu]` → `menu.*`)
- 🥇 **Format Precedence**: Later format wins — json, yaml, toml, csv, tsv, then `.tr` supreme
- 🔢 **CLDR-lite Plurals**: Correct one/few/many/other/zero/two rules for Germanic, Slavic, French, Arabic, Polish, Czech/Slovak, and Asian (no-plural) families
- 🔄 **Pipe Templates**: Positional plural forms (`"one|other"`, `"one|few|many"`) selected by language form order
- 🧩 **`{placeholder}` Interpolation**: String vars maps with automatic `{count}` injection for plurals
- 🖥️ **System Language Detection**: Native locale via the `sysinfo` package with `"en"` fallback
- 🧪 **Test & Benchmark Suite**: 150 assertions (`std/test`) and micro-benchmarks
- 🔍 **Coverage Introspection**: `has` / `missing_keys` translation auditing
- 🔢 **Locale Numbers & Dates**: Per-family decimal/grouping separators, MDY/DMY/YMD numeric dates

---

## 📁 Project Architecture

```
i18n/
├── .alyalint               # Linter configuration (rules, exclusions, severity overrides)
├── .editorconfig           # Uniform formatting rules across IDEs and editors
├── .gitignore              # Ecosystem standard ignore filters
├── .vscode/                # VS Code workspace settings, DAP launch configurations & tasks
├── alya.toml               # Package manifest (depends on `json`, `sysinfo`, `yaml`, `toml`)
├── src/
│   ├── lib.alya            # Public API facade (load_bundle, t, plural, tr_in)
│   ├── types.alya          # I18nBundle model, constructors, set_lang/langs
│   ├── tr_parser.alya      # .tr file parser
│   ├── json_loader.alya    # Flat JSON loader
│   ├── csv_loader.alya     # Minimal CSV/TSV reader (key,value records)
│   ├── yaml_loader.alya    # Flat YAML loader
│   ├── toml_loader.alya    # TOML loader ([table] → dotted keys)
│   ├── plural.alya         # CLDR-lite categories + pipe form orders
│   ├── format.alya         # Placeholder interpolation
│   ├── number.alya         # Locale number/date formatting
│   ├── detect.alya         # System language detection (sysinfo)
│   └── loader.alya         # Directory walker (.tr wins, subdir namespaces)
├── examples/
│   ├── demo.alya           # Runnable walkthrough of all package capabilities
│   └── translations/       # Demo translation files (en.tr, tr.tr)
├── tests/
│   ├── test_basic.alya     # Automated test suite (150 assertions)
│   └── translations/       # Fixture files (en/tr/de, .tr + .json)
└── benches/
    └── bench_basic.alya    # Micro-benchmarks measuring performance and throughput
```

> [!NOTE]
> **Visibility & Modularity:** Symbols annotated with `pub` (`pub function`, `pub struct`, `pub enum`, `pub interface`) are exported to external consumers and re-exporting modules. Symbols without `pub` remain strictly internal to their declaring module, preventing symbol collisions and implementation leakage.

---

## 📦 Installation

Add `i18n` to the `[dependencies]` section in your `alya.toml`:

```toml
[dependencies]
i18n = { git = "https://github.com/alya-lang/i18n", branch = "main" }
```

Or install it directly using the Alya package CLI:

```bash
alya add i18n --git https://github.com/alya-lang/i18n --branch main
alya install
```

---

## 🚀 Quick Start

```alya
import "i18n" as pkg

function main()
    # 1. Inline bundle with plural templates
    let enm = map()
    enm["hello"] = "Hi"
    enm["goods"] = """one item|{{count}} items"""
    let b = pkg::bundle_new("en", "en")
    let b = b.put_lang("en", enm)
    say pkg::t(b, "hello")          # "Hi"
    say pkg::plural(b, "goods", 5)  # "5 items"

    # 2. Load a translation directory (auto-detects system language)
    let lb = pkg::load_bundle("translations", "", "en")
    say pkg::t(lb, "msg_hello")
end

main()
```

> [!NOTE]
> **v0.1.0 contracts:** JSON/YAML/TOML translation files are flat objects of
> string values (literal dotted keys stay literal); nested objects, arrays,
> and non-string scalars are not consumed — quote every value. CSV/TSV files
> use a `key,value` header row. Placeholder vars values must be strings —
> inject counts as `str(n)`. These limits follow from native map subscript
> semantics (variable-key reads of heap values need `str_from_ptr` pinning);
> richer shapes are planned alongside compiler map improvements.

---

## 📖 API Reference

| Symbol | Visibility | Description |
|---|---|---|
| `bundle_new(lang, fallback)` | `pub function` | Creates an empty `I18nBundle` (defaults `"en"`, `"en"`). Fill it with `put_lang`. |
| `I18nBundle.put_lang(lang, langmap)` | `pub method` | Merges a flat key → template map under a language id. |
| `I18nBundle.set_lang(lang)` | `pub method` | Returns bundle copy with updated active language. |
| `I18nBundle.langs()` | `pub method` | Lists language ids present in the bundle. |
| `load_bundle(dir, lang, fallback)` | `pub function` | Loads a translation directory; empty `lang` auto-detects the system language. |
| `load_dir(dir)` | `pub function` | Loads `<lang>.tr` / `<lang>.json` (+ language subdirs) into a flat table (`.tr` wins). |
| `parse_tr_text(text)` | `pub function` | Parses `.tr` text (key lines, `-----` separators, multiline values). |
| `parse_json_text(text)` | `pub function` | Parses flat JSON string values (literal dotted keys stay literal). |
| `parse_csv_text(text)` | `pub function` | Parses `key,value` CSV records (first row is the header). |
| `parse_tsv_text(text)` | `pub function` | Parses `key<tab>value` TSV records. |
| `parse_yaml_text(text)` | `pub function` | Parses flat YAML string mappings. |
| `parse_toml_text(text)` | `pub function` | Parses top-level TOML `key = "value"` pairs. |
| `t(bundle, key, vars)` | `pub function` | Translates with fallback chain; missing keys return the key itself. |
| `plural(bundle, key, n, vars)` | `pub function` | Pipe-template plural with `{count}` injection. |
| `tr_in(bundle, lang, key, vars)` | `pub function` | Translates in an explicit language. |
| `plural_category(lang, n)` | `pub function` | CLDR-lite category (`zero/one/two/few/many/other`). |
| `form_order(lang)` | `pub function` | Ordered form keys for pipe templates. |
| `base_lang(tag)` | `pub function` | Base language (`"pt-BR"` → `"pt"`). |
| `interpolate(template, vars)` | `pub function` | Replaces `{key}` placeholders from a string vars map. |
| `detect_lang(default)` | `pub function` | System locale tag via `sysinfo` (`"en"` fallback). |
| `system_language(default)` | `pub function` | Alias of `detect_lang`. |
| `find_raw(bundle, lang, key)` | `pub function` | Raw template lookup (null when untranslated). |
| `has(bundle, key)` | `pub function` | Reports whether key resolves in the language chain. |
| `missing_keys(bundle, lang)` | `pub function` | Keys translated in fallback but missing in lang. |
| `plural_in(bundle, key, n, lang, vars)` | `pub function` | Plural translation in an explicit language. |
| `month_name(lang, month)` | `pub function` | Full month name (12 languages, English via stdlib). |
| `month_short(lang, month)` | `pub function` | Abbreviated month name. |
| `format_date_named(year, month, day, lang)` | `pub function` | Locale-ordered date with month name. |
| `weekday_name(lang, weekday)` | `pub function` | Full weekday name, Sunday is 0 (English via stdlib). |
| `weekday_short(lang, weekday)` | `pub function` | Abbreviated weekday name. |
| `format_date_full(year, month, day, lang)` | `pub function` | Full date with weekday (Zeller congruence). |
| `format_time(hour, minute, lang)` | `pub function` | Locale time (12h with markers or 24h). |
| `relative_past(bundle, lang, n, unit)` | `pub function` | Localized "ago" time from bundle templates. |
| `relative_future(bundle, lang, n, unit)` | `pub function` | Localized "in" time from bundle templates. |
| `format_money(value, lang, currency, decimals)` | `pub function` | Currency with locale separators and symbol position. |
| `weekday_name(lang, weekday)` | `pub function` | Full weekday name, Sunday is 0 (English via stdlib). |
| `weekday_short(lang, weekday)` | `pub function` | Abbreviated weekday name. |
| `format_date_full(year, month, day, lang)` | `pub function` | Full date with weekday (Zeller congruence). |
| `fallback_chain(bundle, lang)` | `pub function` | Deduplicated lookup chain. |
| `vars_with_count(vars, n)` | `pub function` | Copies string vars and injects `{count}`. |
| `I18nBundle` | `pub struct` | Bundle model (`messages`, `lang`, `fallback`). |

> [!TIP]
> **Internal Helpers & Documentation:** Public symbols are documented with `##` Markdown docstrings, enabling automatic API documentation generation via `alya doc`. Module-local helpers remain encapsulated without `pub`.

---

## 🧪 Running Tests & Benchmarks

Run the automated test suite using `alya test`:

```bash
alya test
```

Generate static API documentation:

```bash
alya doc . -o docs --markdown
```

Run the benchmark suite:

```bash
alya run benches/bench_basic.alya
```

Run the example demo:

```bash
alya run examples/demo.alya
```

Check code formatting:

```bash
alya fmt . --check
```

Run static code linter:

```bash
alya lint . --check
```

---

### 💻 Developer Tooling & VS Code Integration

This package comes preconfigured with recommended workspace settings and tasks for **Visual Studio Code**:
- **LSP & Formatting**: Auto-formatting on save and real-time Language Server diagnostics via `alya-lang.vscode-alya`.
- **DAP Debugging**: Launch configurations in `.vscode/launch.json` ready for interactive step-debugging via `F5`.
- **Predefined Tasks**: Press `Ctrl+Shift+B` or run tasks (`Test`, `Lint`, `Format`, `Build Docs`) directly from the Command Palette.

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install dependencies:
   ```bash
   alya install
   ```
3. Create your feature branch (`git checkout -b feature/my-feature`)
4. Verify tests and formatting before opening a PR:
   ```bash
   alya test
   ```
5. Commit your changes (`git commit -m "feat: add feature"`) and open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
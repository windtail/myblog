---
title: "Tauri发布时手动下载nsis"
date: 2025-08-16T10:17:55+08:00
draft: false
tags: ["rust", "tauri", "bundle", "nsis"]
---

## 问题

[Tauri](https://tauri.app) 是个很好的东西，但是Tauri打包的时候会自动下载nsis，离线的时候，下载会失败，手动下载放到哪里呢？
这次记录下，省得下次再忘记了。

## 源代码分析

查看 [tauri-app/tauri](https://github.com/tauri-apps/tauri) 仓库里，找到 [bundle_project](https://github.com/tauri-apps/tauri/blob/21ebc6e82062b55a12f3a360d9a979daf5ae7e66/crates/tauri-bundler/src/bundle/windows/nsis/mod.rs#L77) 函数即可，
参见如下代码：

```rust
  let tauri_tools_path = settings
    .local_tools_directory()
    .map(|d| d.join(".tauri"))
    .unwrap_or_else(|| dirs::cache_dir().unwrap().join("tauri"));

  let nsis_toolset_path = tauri_tools_path.join("NSIS");

  if !nsis_toolset_path.exists() {
    get_and_extract_nsis(&nsis_toolset_path, &tauri_tools_path)?;
    
    
fn get_and_extract_nsis(nsis_toolset_path: &Path, _tauri_tools_path: &Path) -> crate::Result<()> {
  log::info!("Verifying NSIS package");

  #[cfg(target_os = "windows")]
  {
    let data = download_and_verify(NSIS_URL, NSIS_SHA1, HashAlgorithm::Sha1)?;
    log::info!("extracting NSIS");
    crate::utils::http_utils::extract_zip(&data, _tauri_tools_path)?;
    fs::rename(_tauri_tools_path.join("nsis-3.08"), nsis_toolset_path)?;
  }

  // download additional plugins
  let nsis_plugins = nsis_toolset_path.join("Plugins");

  let data = download_and_verify(
    NSIS_TAURI_UTILS_URL,
    NSIS_TAURI_UTILS_SHA1,
    HashAlgorithm::Sha1,
  )?;

  let target_folder = nsis_plugins.join("x86-unicode").join("additional");
  fs::create_dir_all(&target_folder)?;
  fs::write(target_folder.join("nsis_tauri_utils.dll"), data)?;

  Ok(())
}  

#[cfg(target_os = "windows")]
const NSIS_URL: &str =
  "https://github.com/tauri-apps/binary-releases/releases/download/nsis-3/nsis-3.zip";
#[cfg(target_os = "windows")]
const NSIS_SHA1: &str = "057e83c7d82462ec394af76c87d06733605543d4";
const NSIS_TAURI_UTILS_URL: &str =
  "https://github.com/tauri-apps/nsis-tauri-utils/releases/download/nsis_tauri_utils-v0.5.1/nsis_tauri_utils.dll";
const NSIS_TAURI_UTILS_SHA1: &str = "B053B2E5FDB97257954C8F935D80964F056520AE";
  
```

在没有配置本地工具文件夹（我还不知道怎么配置）时，系统目录在 `dirs::cache_dir()` 下面的 `tauri` 子目录，下载 [nsis](https://github.com/tauri-apps/binary-releases/releases/download/nsis-3/nsis-3.zip) 到 `tauri` 目录中，
解压成 `nsis-3.0.8`，然后重命名为 `NSIS`，最后再下载 [nsis_tauri_utils.dll](https://github.com/tauri-apps/nsis-tauri-utils/releases/download/nsis_tauri_utils-v0.5.1/nsis_tauri_utils.dll) 到 `NSIS/Plugins/x86-unicode/additional/nsis_tauri_utils.dll`

至于`dirs::cache_dir()`，这个函数的代码如下：

```rust
/// Returns the path to the user's cache directory.
///
/// ## Platform-specific
///
/// - **Linux:** Resolves to `$XDG_CACHE_HOME` or `$HOME/.cache`.
/// - **macOS:** Resolves to `$HOME/Library/Caches`.
/// - **Windows:** Resolves to `{FOLDERID_LocalAppData}`.
pub fn cache_dir(&self) -> Result<PathBuf> {
    dirs::cache_dir().ok_or(Error::UnknownPath)
}
```

## 总结

Windows 在 `%USERPROFILE%\AppData\Local` 下，也就是最后 nsis 在 `%USERPROFILE%\AppData\Local\tauri\NSIS` 下面。
Wix应该以是类似的，暂时用不上，没有研究。

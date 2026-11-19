# Flutter iOS Cloud Build

用 GitHub Actions（macOS runner）云构建 Flutter iOS IPA，无需本地 Mac。

## 为什么这样设计
iOS 编译/签名工具链需要 macOS + Xcode。本项目在 Windows 上写 Dart 代码，推到 GitHub 后由 `macos-14` runner 完成构建。
仓库只存文本文件。iOS 原生目录（`ios/`，含二进制图标）由 CI 里的 `flutter create --platforms ios` 现场生成，绕开二进制文件入库问题。

## 工作流
`.github/workflows/ios-build.yml` 触发时机：push/PR 到 main 或 master，也可在 Actions 页面手动 `workflow_dispatch`。
步骤：checkout → setup Flutter 3.50.0 → `flutter create` 生成 iOS 脚手架 → `pub get` → `analyze` → `test` → `flutter build ipa --no-codesign` → 上传 IPA artifact（保留 30 天）。

## 本地开发（Windows）
```powershell
flutter pub get
flutter analyze
flutter test
flutter run -d chrome      # 用 Web 验证 UI
```

## 出包说明
`--no-codesign` 生成的是未签名 IPA，用于验证构建流程。要真机安装或上架：
- 加入 Apple Developer 账号（$99/年）
- 在 workflow 里配置签名证书 + Provisioning Profile（可用 `apple-actions/import-codesign-certs`）
- 改用 `flutter build ipa --export-options-plist` 导出签名包

## 换自己的 app
直接用你自己的 Flutter 项目替换本仓库的 `lib/`、`test/`、`pubspec.yaml`、`analysis_options.yaml` 即可；若你已把完整 `ios/` 目录入库，可删掉 workflow 里的 `flutter create` 步骤。

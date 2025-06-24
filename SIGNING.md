# 📱 Android APK 签名配置说明

## 🔐 签名策略

这个项目采用**智能签名策略**，确保无论是否配置了正式签名密钥，都能成功构建签名的APK：

### ✅ 模式一：正式签名（推荐生产环境）
- **条件**：配置了完整的签名密钥secrets
- **结果**：使用你的正式签名密钥构建APK
- **用途**：可发布到应用商店或正式分发

### ⚠️ 模式二：测试签名（开发/测试环境）
- **条件**：未配置或配置不完整的签名密钥
- **结果**：自动创建临时测试密钥并签名APK
- **用途**：仅供开发测试使用

## 🔑 配置正式签名密钥

要启用正式签名，需要在GitHub仓库的 **Settings → Secrets and variables → Actions** 中设置以下4个secrets：

| Secret名称 | 说明 | 示例 |
|-----------|------|------|
| `KEY_JKS` | base64编码的密钥文件内容 | `MIIEvgIBADANBgkqhkiG9w0B...` |
| `ALIAS` | 密钥别名 | `release-key` |
| `ANDROID_STORE_PASSWORD` | 密钥库密码 | `your_store_password` |
| `ANDROID_KEY_PASSWORD` | 密钥密码 | `your_key_password` |

### 📝 如何获取密钥文件的base64内容：

```bash
# 在macOS/Linux上
base64 -i your-keystore.jks | pbcopy

# 在Windows上（使用PowerShell）
[Convert]::ToBase64String([IO.File]::ReadAllBytes("your-keystore.jks")) | Set-Clipboard
```

## 🛠️ 创建新的签名密钥

如果你还没有签名密钥，可以使用以下命令创建：

```bash
keytool -genkey -v -keystore release-key.jks \
  -alias release-key \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -storepass YOUR_STORE_PASSWORD \
  -keypass YOUR_KEY_PASSWORD \
  -dname "CN=Your Name, OU=Your Unit, O=Your Organization, L=Your City, S=Your State, C=Your Country"
```

## 🚀 构建流程

### 检查密钥配置
workflow会自动检查是否配置了完整的签名密钥：

```
✅ 所有签名密钥已配置，将使用提供的密钥构建签名APK
或
⚠️ 签名密钥未完整配置，将使用临时测试密钥构建APK
```

### 密钥文件设置
- **有配置**：从secrets解码base64内容创建密钥文件
- **无配置**：使用keytool创建临时测试密钥

### 验证密钥
workflow会自动验证密钥文件：
- 列出密钥文件中的所有别名
- 检查密钥密码是否正确
- 显示详细的调试信息

### 构建签名APK
使用正确的密钥参数构建签名的APK文件

## 🔒 安全注意事项

1. **保护你的签名密钥**：
   - 不要将密钥文件提交到代码仓库
   - 使用强密码保护密钥
   - 定期备份你的签名密钥

2. **Secrets安全**：
   - GitHub Secrets是加密存储的
   - 只有有权限的用户才能查看和修改
   - 构建日志中不会显示密钥内容

3. **测试密钥限制**：
   - 临时测试密钥每次构建都会重新生成
   - 不要用测试签名的APK进行正式发布
   - 测试APK无法升级到正式签名的版本

## 📱 APK识别

构建完成后，Release页面会明确标识APK类型：

- ✅ **正式签名APK** - 使用配置的生产密钥签名，可直接安装使用
- ⚠️ **测试签名APK** - 使用临时测试密钥签名，仅供测试使用

## ❓ 常见问题

### Q: 为什么要使用测试签名？
A: 确保即使没有配置正式密钥，也能成功构建APK进行测试。

### Q: 测试签名的APK能否用于生产？
A: 不建议。测试签名APK仅用于开发测试，不能升级到正式签名版本。

### Q: 如何从测试签名切换到正式签名？
A: 配置完整的4个signing secrets即可自动切换到正式签名模式。

### Q: 构建失败怎么办？
A: 检查GitHub Actions日志，workflow会显示详细的调试信息。 
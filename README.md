# anitabi-cutout-runtime

[动画巡礼 Android 版](https://github.com/AbuCuma/anitabi-android-app)「AI 抠图增强」实验功能的
运行时分发点。这个仓库**只放二进制**,没有源码。

App 里这个功能**默认关闭**,需要用户在「关于」页手动打开;打开后进入对比拍摄时才会在
**非计费网络**下载下面这些文件。不开启的话完全不会访问这里。

## 内容

| 文件 | 大小 | 许可 | 来源 |
|---|---|---|---|
| `isnet_w8a8.onnx` | 42.4 MB | Apache-2.0 | [SkyTNT/anime-segmentation](https://github.com/SkyTNT/anime-segmentation) 的 w8a8 量化产物 |
| `libonnxruntime.so` | 22.9 MB | MIT | [microsoft/onnxruntime](https://github.com/microsoft/onnxruntime) `onnxruntime-android-qnn-1.29.0.aar` 的 `jni/arm64-v8a/` |
| `manifest.json` | — | — | 大小与 SHA-256 清单,由 `tools/build_cutout_manifest.py` 生成 |

App 侧按 manifest 校验**大小 + SHA-256**,通过后才 `setReadOnly` 并写入 `.ok` 标记;
校验不过就删掉重来,或者降级回 Google ML Kit。

## 为什么只有 CPU 档

ISNet 在 App 里有 HTP(高通 NPU,20〜30ms)与 CPU(约 0.7s)两档。HTP 档需要 Qualcomm 的
QNN/QAIRT 运行库,而 Qualcomm AI Stack License 授予的是「以 object code 形式、**且在集成进
你的软件应用的前提下**」分发的权利,**未**授予 standalone 分发。把 `libQnn*.so` 作为独立
下载资产放在这里正属于后者,所以这里不提供。

缺件时 App 的 `CutoutManifest.requiredArtifacts` 返回 null,`CutoutEngine` 自动降到 CPU 档 ——
这是既有的降级路径,不需要特殊处理。

细节见 App 仓库的 [docs/CUTOUT_RUNTIME.md](https://github.com/AbuCuma/anitabi-android-app/blob/main/docs/CUTOUT_RUNTIME.md)。

## 版本一致性

`libonnxruntime.so` 的版本必须与 App 侧 ORT 的 Java 依赖版本**严格一致**(当前 1.29.0),
否则运行时崩溃。升级 ORT 依赖时必须同步在这里发一个新 tag 并更新 App 里的 manifest URL。

## 许可

本仓库分发的文件各自保留其上游许可证(见上表),此处不做任何再授权。

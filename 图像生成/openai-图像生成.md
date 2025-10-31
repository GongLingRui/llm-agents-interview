# OpenAI API 图像生成功能网页总结
## 一、核心API选择
OpenAI提供两种图像生成相关API，适用于不同场景，且均支持自定义输出（调整质量、尺寸、格式等）。

| API类型 | 核心能力 | 支持模型 | 适用场景 |
| ---- | ---- | ---- | ---- |
| Image API | 3个独立端点：<br>- Generations：基于文本从零生成图像<br>- Edits：通过新提示修改现有图像（含蒙版修图）<br>- Variations：生成图像变体（仅DALL·E 2支持） | gpt-image-1、dall-e-2、dall-e-3 | 单步图像生成/编辑，无需多轮交互 |
| Responses API | 作为对话或多步流程的内置工具，支持：<br>- 多轮迭代编辑图像<br>- 接受File ID作为输入（非仅字节） | 仅gpt-image-1（需搭配gpt-4o及以上主模型） | 构建对话式图像体验，需多轮优化图像 |


## 二、支持模型对比
不同模型功能侧重不同，gpt-image-1为最新且功能最全面的模型。

| 模型 | 支持端点 | 核心优势 | 适用场景 |
| ---- | ---- | ---- | ---- |
| DALL·E 2 | Image API：Generations、Edits、Variations | 成本低、支持并发请求、蒙版修图（inpainting） | 基础图像生成/变体，追求性价比 |
| DALL·E 3 | Image API：仅Generations | 图像质量高于DALL·E 2，支持更大分辨率 | 单步高质量图像生成 |
| GPT Image（gpt-image-1） | Image API：Generations、Edits；Responses API（即将支持） | 指令遵循性强、文本渲染优、细节编辑精准、可结合真实世界知识 | 复杂编辑、多轮交互、高保真图像生成 |

> 注：使用gpt-image-1需先在开发者控制台完成“API组织验证”。


## 三、核心功能操作指南
### （一）图像生成
1. **基础生成**：通过API指定模型、文本提示，可设置`n`参数生成多张图像（默认1张），支持Python、Node.js、curl等代码调用。
   - 示例：生成“戴橙色围巾的灰色虎斑猫拥抱水獭”图像，通过Base64编码保存为本地文件。
2. **多轮生成**（仅Responses API支持）：
   - 方式1：使用`previous_response_id`参数，基于上一轮生成结果优化。
   - 方式2：传入图像ID，结合新提示迭代修改。
3. **流式生成**：两种API均支持，通过`stream=true`和`partial_images`（0-3张）参数，流式返回部分图像，提升交互体验。
   - 示例：生成“猫头鹰羽毛组成的河流”图像，设置`partial_images=2`，分阶段获取2张部分图像和1张最终图像。
4. **提示词优化**（仅Responses API）：主模型（如gpt-4.1）会自动优化输入提示，优化后内容可在`revised_prompt`字段查看。


### （二）图像编辑
1. **基于参考图生成新图**：
   - 支持传入多张参考图（Base64编码或File ID），生成包含参考元素的新图像。
   - 示例：传入身体乳、香皂、香薰等参考图，生成“Relax & Unwind”主题礼品篮图像。
2. **蒙版修图（Inpainting）**：
   - 提供蒙版图像（需与原图同尺寸、含Alpha通道），指定修改区域，结合提示生成新图像。
   - 差异：GPT Image基于提示灵活调整蒙版区域，无需严格遵循蒙版形状；DALL·E 2严格按蒙版修改。
3. **高输入保真度**：
   - 设置`input_fidelity="high"`，可精准保留输入图像细节（如人脸、logo），首图细节保留最完整。
   - 注意：会增加输入图像token消耗，需参考“成本与延迟”部分计算费用。


### （三）输出自定义
可通过参数配置图像输出属性，部分参数支持`auto`模式（模型自动匹配最优值）。

| 配置项 | 可选值 | 说明 |
| ---- | ---- | ---- |
| 尺寸（size） | 1024x1024（正方形）、1024x1536（竖版）、1536x1024（横版）、auto（默认） | 正方形+标准质量生成速度最快，默认1024x1024 |
| 质量（quality） | low、medium、high、auto（默认） | 质量越高，token消耗越多，生成时间越长 |
| 格式（format） | png（默认）、jpeg、webp | jpeg生成速度快，适合低延迟需求；png/webp支持透明背景 |
| 压缩（compression） | 0-100%（仅jpeg/webp支持） | 示例：`output_compression=50`表示压缩50% |
| 背景（background） | transparent（透明）、opaque（不透明）、auto | 仅png/webp支持透明背景，建议搭配medium/high质量 |


## 四、限制与内容审核
1. **模型限制**：
   - 延迟：复杂提示可能需2分钟处理；
   - 文本渲染：虽优于DALL·E系列，但精准度仍有限；
   - 一致性：多轮生成中，重复角色/品牌元素的视觉一致性可能不稳定；
   - 构图控制：结构化布局中，元素精准放置难度较高。
2. **内容审核**：
   - 所有提示和生成图像均按OpenAI内容政策过滤；
   - gpt-image-1支持`moderation`参数调整严格度：`auto`（默认，标准过滤）、`low`（宽松过滤）。
3. **模型兼容性**：Responses API需搭配gpt-4o及以上主模型，具体可查看模型详情页确认支持性。


## 五、成本与延迟计算
成本由“输入文本token+输入图像token（编辑时）+输出图像token”三部分构成，延迟与token数量正相关（token越多，延迟越高）。

### （一）输出图像token消耗（按尺寸和质量）
| 质量 | 正方形（1024×1024） | 竖版（1024×1536） | 横版（1536×1024） |
| ---- | ---- | ---- | ---- |
| Low | 272 token | 408 token | 400 token |
| Medium | 1056 token | 1584 token | 1568 token |
| High | 4160 token | 6240 token | 6208 token |

### （二）额外成本
- 高输入保真度：增加输入图像token消耗；
- 流式生成（partial_images）：每张部分图像额外消耗100 token；
- 输入token：文本提示按文本token计费，编辑时输入图像按图像token计费。

> 具体单价可参考OpenAI定价页面，需结合实际使用场景计算总成本。



Variations: Generate variations of an existing image (available with DALL·E 2 only)
变体：生成现有图像的变体（仅DALL·E 2支持）

Multi-turn editing: Iteratively make high fidelity edits to images with prompting

多轮编辑：通过提示词对图像进行迭代式高保真编辑

Flexible inputs: Accept image File IDs as input images, not just bytes

灵活的输入：接受图像文件ID作为输入图像，而不仅仅是字节

<img width="665" height="356" alt="image" src="https://github.com/user-attachments/assets/029d6cf2-33ef-42d9-8962-ab366bf1c603" />


You can set the n parameter to generate multiple images at once in a single request (by default, the API returns a single image).
你可以设置n参数，在一次请求中同时生成多张图片（默认情况下，API返回一张图片）。

With the Responses API, you can build multi-turn conversations involving image generation either by providing image generation calls outputs within context (you can also just use the image ID), or by using the
借助响应API，你可以构建涉及图像生成的多轮对话，具体方式有两种：一是在上下文中提供图像生成调用的输出（你也可以只使用图像ID），二是通过使用

在Responses API中使用图像生成工具时，主线模型（例如gpt-4.1）会自动修改您的提示词以提升性能。

You can access the revised prompt in the revised_prompt field of the image generation call:
你可以在图像生成调用的revised_prompt字段中访问修改后的提示词

With the Responses API, you can provide input images in 2 different ways:
借助Responses API，您可以通过两种不同方式提供输入图像：

By providing an image as a Base64-encoded data URL
通过提供作为Base64编码数据URL的图像
By providing a file ID (created with the Files API)
通过提供文件ID（使用文件API创建）

You can provide multiple input images that will all be preserved with high fidelity, but keep in mind that the first image will be preserved with richer textures and finer details, so if you include elements such as faces, consider placing them in the first image.
你可以提供多张输入图像，所有图像都将以高保真度保留，但请记住，第一张图像将保留更丰富的纹理和更精细的细节，因此如果你要包含人脸等元素，请考虑将它们放在第一张图像中。


Customize Image Output 自定义图像输出
You can configure the following output options:
您可以配置以下输出选项：

Size: Image dimensions (e.g., 1024x1024, 1024x1536)
尺寸：图像尺寸（例如，1024x1024、1024x1536）
Quality: Rendering quality (e.g. low, medium, high)
质量：渲染质量（例如low、medium、high）
Format: File output format 格式：文件输出格式
Compression: Compression level (0-100%) for JPEG and WebP formats
压缩：JPEG和WebP格式的压缩级别（0-100%）
Background: Transparent or opaque 背景：透明或不透明
size, quality, and background support the auto option, where the model will automatically select the best option based on the prompt.
size、quality和background支持auto选项，模型会根据提示词自动选择最佳选项。

The Image API returns base64-encoded image data. The default format is png, but you can also request jpeg or webp.
图像API返回经过base64编码的图像数据。默认格式为png，但你也可以请求jpeg或webp格式。

If using jpeg or webp, you can also specify the output_compression parameter to control the compression level (0-100%). For example, output_compression=50 will compress the image by 50%.
如果使用jpeg或webp</b1，还可以指定output_compression参数来控制压缩级别（0-100%）。例如，output_compression=50会将图像压缩50%。


Using jpeg is faster than png, so you should prioritize this format if latency is a concern.
使用jpeg比使用png更快，因此如果担心延迟问题，你应该优先选择这种格式。

Note that you will also need to account for input tokens: text tokens for the prompt and image tokens for the input images if editing images. If you are using high input fidelity, the number of input tokens will be higher.
请注意，您还需要考虑输入 tokens：如果是编辑图像，包括提示词的文本 tokens 和输入图像的图像 tokens。如果您使用高输入保真度，输入 tokens 的数量会更多。


f you want to stream image generation using the partial_images parameter, each partial image will incur an additional 100 image output tokens.
如果您想使用partial_images参数来流式生成图像，每个部分图像将额外消耗100个图像输出令牌。


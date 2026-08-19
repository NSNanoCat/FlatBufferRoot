# @nsnanocat/flatbuffer-root

基于 FlatBuffers JavaScript 生成模型，按根表 slot 解码、修改并重新组装 FlatBuffer。处理器只解析显式注册的产品表，其余已知字段和未来 schema 新增的未知 slot 会保持原始二进制内容。

Decode, patch, and reassemble FlatBuffers root-table slots with generated JavaScript models. The processor only parses explicitly registered product tables and preserves the original binary data for all other known or future slots.

## Install

```shell
npm install @nsnanocat/flatbuffer-root flatbuffers
```

`flatbuffers` 是 peer dependency，支持 `>=24.12.23 <26`。

`flatbuffers` is a peer dependency. Supported versions are `>=24.12.23 <26`.

## Processing Model

处理器从生成的根表类推导 slot 名称和顺序，并为需要读写的 slot 注册 codec：

- `rootClass.prototype` 上除 `constructor` 和 `__init` 外的方法会按声明顺序映射到物理 slot。
- 每个根 accessor 都必须存在对应的静态 `add{Name}` 方法。
- 根字段必须是 table offset；scalar、string、vector 等根字段不在支持范围内。
- 未注册 codec 的 schema slot 不会被解析，但使用源 FlatBuffer 编码时仍会透明保留。
- 超出当前 schema 的物理 slot 同样会作为 opaque arena 保留。

The processor derives slot names and order from the generated root-table class and registers codecs only for slots that need to be read or written:

- Methods on `rootClass.prototype`, except `constructor` and `__init`, map to physical slots in declaration order.
- Every root accessor must have a matching static `add{Name}` method.
- Root fields must be table offsets. Scalar, string, vector, and other root field types are not supported.
- Schema slots without a codec are not parsed, but remain intact when encoding from a source FlatBuffer.
- Physical slots beyond the current schema are also preserved as opaque arenas.

## Usage

```js
import { FlatBufferRootProcessor } from "@nsnanocat/flatbuffer-root";
import { ByteBuffer } from "flatbuffers";

const processor = new FlatBufferRootProcessor({
    name: "Example",
    rootClass: ExampleRoot,
    codecs: {
        product: {
            tableClass: ExampleProduct,
            decode: table => decodeProduct(table),
            encode: (builder, json) => encodeProduct(builder, json),
        },
    },
    configurableRootNames: ["product"],
});

const source = new ByteBuffer(rawBody);
const rootNames = processor.filterRootNames(requestedRootNames, enabledRootNames);
const json = processor.decode(source, rootNames);
const output = processor.encode(source, {
    product: updateProduct(json.product),
});
```

`name` 用作日志前缀。`rootClass`、`tableClass` 以及 codec 名称在构造时一次性校验；无效配置会立即抛出异常。

`name` is used as the diagnostic prefix. `rootClass`, `tableClass`, and codec names are validated once during construction, and invalid configurations fail immediately.

## API

### `filterRootNames(requestedRootNames, enabledRootNames)`

仅过滤 `configurableRootNames` 中受业务开关控制的名称，并保持请求的原始顺序。未列为 configurable 的名称会原样保留。

Filters only names controlled by `configurableRootNames` while preserving request order. Names that are not configurable pass through unchanged.

### `decode(byteBuffer, rootNames = [])`

按物理 slot 顺序解码已请求且已注册 codec 的产品表，返回成功解码的部分对象。未知名称、缺失 slot 和可隔离的单 slot 失败会写入诊断日志；无效根表或越界数据会抛出异常。

Decodes requested product tables with registered codecs in physical slot order and returns the successfully decoded partial object. Unknown names, missing slots, and isolated per-slot failures are reported through diagnostics; an invalid root table or out-of-range data throws.

### `encode(byteBuffer = undefined, patch = {})`

每个 patch slot 会独立编译。成功的 slot 会替换源数据中的对应 slot，失败或未知的 patch 不会覆盖原值；未修改和未知的源 slot 会保留。省略 `byteBuffer` 时会创建只包含成功 patch 的新根表。

Each patch slot is compiled independently. Successful slots replace their source counterparts, while failed or unknown patches do not overwrite the original data. Unmodified and unknown source slots remain intact. Omitting `byteBuffer` creates a new root table containing only successful patches.

## Diagnostics

配置错误和不可恢复的输入错误通过 `@nsnanocat/util` 的 `Console.error` 记录后抛出。可隔离的未知、缺失或失败 slot 使用 `Console.warn`，完整成功的阶段统计使用 `Console.debug`。

Configuration errors and unrecoverable input errors are logged through `Console.error` from `@nsnanocat/util` before being thrown. Isolated unknown, missing, or failed slots use `Console.warn`; successful phase summaries use `Console.debug`.

## Development

```shell
npm ci
npm run format
npm run lint
npm test
npm run typecheck
npm run check
npm pack --dry-run
```

推送 `v*` tag 后，GitHub Actions 会先执行安装和测试，再分别发布到 npm 与 GitHub Packages。

Pushing a `v*` tag runs installation and tests in GitHub Actions before publishing to npm and GitHub Packages.

## License

[Apache License 2.0](LICENSE)

# SQLite Vector Embedding Database Plugin (UE5)

轻量级 **SQLite** 数据库插件，**内置并固定集成** `sqlite-vec` 向量检索扩展。
本插件为**纯代码插件**（不含内容资产），用于在 Unreal Engine 中执行 SQL 与向量相似度检索（AI 记忆 / Embedding 检索等场景）。

- **Publisher:** haozena  
- **Supported UE:** 5.6（Win64）  
- **Module:** `SQLiteVecPlugin`  
- **Third-party:** SQLiteCore（核心，必需），SQLiteVec（必需，随插件集成，动态加载）  
- **Content:** `"CanContainContent": false`

---

## 1) 安装 Installation

1. 关闭 Unreal Editor。  
2. 将整个 `SQLiteVecPlugin/` 文件夹复制到：  
   - **推荐**：`[YourProject]/Plugins/`，或  
   - `Engine/Plugins/`（全局安装）。
3. 确认第三方依赖随包存在：  
   - `Source/ThirdParty/Include/` → `sqlite3.h`, `sqlite3ext.h`, `sqlite-vec.h`  
   - `Source/ThirdParty/Bin/` → `SQLiteCore.dll`, `SQLiteVec.dll`与各自 **LICENSE**
4. 启动 UE → **Edit > Plugins** → 勾选 **SQLite Vector Embedding Database Plugin**。  
5. 首次打包编译/运行时，`Build.cs` 会把所需 DLL 复制到运行目录（通常 `YourProject/Binaries/Win64`）。



---

## 2) 快速开始（Blueprint 工作流）

> 该插件已在 `StartupModule` 中自动注册/加载 `sqlite-vec` 扩展。下面所有蓝图节点均来自本插件。

### 步骤 A：打开数据库
- **OpenDatabase(FilePath) → DBHandle(int32)**  
  例：`Saved/SQLite/demo.db`（不存在会自动创建）。

### 步骤 B：创建向量表（**Create SQLite Vec Table if not exists**）
- 调用 **CreateVecTable(DBHandle, OutError, TableName="items", Dim=1536)**。  
  这会创建 `vec0` 虚表，向量列名为 `embedding`（内部实现）。

### 步骤 C：写入向量（**Insert Vec Row**）
- 调用 **InsertVecRow(DBHandle, TableName, RowId, VecData(float[]), OutError)**。  
  - 向量需与建表时的 `Dim` 一致；可先用 **PadFloatArray(InArray, TargetLength=Dim, FillValue=0.0)** 进行补零/截断。

### 步骤 D：相似度检索（**Vector Query (K-NN)**）
- 调用 **VectorQuery(DBHandle, TableName, QueryVec(float[]), K, OutResults, OutError)**。  
  - `OutResults : TArray<FVectorSearchResult>`，含 `Id` 与 `Distance`。

### 步骤 E：关闭数据库
- **CloseDatabase(DBHandle)**

> 普通 SQL 可使用 **ExecuteSQL** / **QuerySQL** 节点。


---

## 3) C++ 示例（与蓝图等价调用）

```cpp
#include "SQLiteVecBPLibrary.h"

int32 Handle = USQLiteVecBPLibrary::OpenDatabase(TEXT("Saved/SQLite/demo.db"));

FString Err;
bool Ok = false;

// A. 创建向量表（注意参数顺序：DBHandle, OutError, TableName, Dim）
Ok = USQLiteVecBPLibrary::CreateVecTable(
    Handle, Err, TEXT("items"), 1536);

// B. 插入一条向量
TArray<float> Vec; Vec.SetNum(1536);
// ... 填充 Vec ...
Ok = USQLiteVecBPLibrary::InsertVecRow(Handle, TEXT("items"), /*RowId=*/1, Vec, Err);

// C. K-NN 检索
TArray<float> Query; Query.SetNum(1536);
// ... 填充 Query ...
TArray<FVectorSearchResult> Results;
Ok = USQLiteVecBPLibrary::VectorQuery(Handle, TEXT("items"), Query, /*K=*/5, Results, Err);

// 关闭数据库
USQLiteVecBPLibrary::CloseDatabase(Handle);
```

© 2025 **haozena**. All Rights Reserved.

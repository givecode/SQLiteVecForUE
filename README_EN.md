# SQLite Vector Embedding Database Plugin (UE5)

A lightweight **SQLite** plugin for Unreal Engine that **bundles and requires** the `sqlite-vec` extension for vector similarity search.  
It’s a **code-only plugin** (no Content assets) for running SQL and K-NN vector search inside UE (AI memory / embedding retrieval, etc.).

- **Publisher:** haozena  
- **Supported UE:** 5.6 (Win64)  
- **Module:** `SQLiteVecPlugin`  
- **Third-party:** SQLiteCore (core, required), SQLiteVec (required; integrated with the plugin and loaded dynamically)  
- **Content:** `"CanContainContent": false`

---

## 1) Installation

1. Close Unreal Editor.  
2. Copy the entire `SQLiteVecPlugin/` folder to:
   - **Recommended:** `[YourProject]/Plugins/`, or  
   - `Engine/Plugins/` (engine-wide install).
3. Verify required third-party files are included with the plugin:
   - `Source/ThirdParty/Include/` → `sqlite3.h`, `sqlite3ext.h`, `sqlite-vec.h`  
   - `Source/ThirdParty/Bin/` → `SQLiteCore.dll`, `SQLiteVec.dll` and their **LICENSE** files
4. Launch UE → **Edit > Plugins** → enable **SQLite Vector Embedding Database Plugin**.  
5. On first package/build/run, the `Build.cs` copies the needed DLLs to the runtime folder (usually `YourProject/Binaries/Win64`).

---

## 2) Quick Start (Blueprint workflow)

> The plugin auto-registers/loads the `sqlite-vec` extension in `StartupModule`. All nodes below are provided by this plugin.

### Step A: Open a database
- **OpenDatabase(FilePath) → DBHandle (int32)**  
  Example: `Saved/SQLite/demo.db` (created automatically if missing).

### Step B: Create a vector table (**Create SQLite Vec Table if not exists**)
- Call **CreateVecTable(DBHandle, OutError, TableName="items", Dim=1536)**.  
  This creates a `vec0` virtual table; the vector column name is `embedding` (handled internally).

### Step C: Insert a vector (**Insert Vec Row**)
- Call **InsertVecRow(DBHandle, TableName, RowId, VecData(float[]), OutError)**.  
  - The vector length must match the `Dim` used at table creation; use **PadFloatArray(InArray, TargetLength=Dim, FillValue=0.0)** to pad/truncate if needed.

### Step D: Similarity search (**Vector Query (K-NN)**)
- Call **VectorQuery(DBHandle, TableName, QueryVec(float[]), K, OutResults, OutError)**.  
  - `OutResults : TArray<FVectorSearchResult>` contains `Id` and `Distance`.

### Step E: Close the database
- **CloseDatabase(DBHandle)**

> For regular SQL, use **ExecuteSQL** / **QuerySQL**.

---

## 3) C++ example (equivalent to the Blueprint flow)

```cpp
#include "SQLiteVecBPLibrary.h"

int32 Handle = USQLiteVecBPLibrary::OpenDatabase(TEXT("Saved/SQLite/demo.db"));

FString Err;
bool Ok = false;

// A. Create vector table (note param order: DBHandle, OutError, TableName, Dim)
Ok = USQLiteVecBPLibrary::CreateVecTable(
    Handle, Err, TEXT("items"), 1536);

// B. Insert a vector row
TArray<float> Vec; Vec.SetNum(1536);
// ... fill Vec ...
Ok = USQLiteVecBPLibrary::InsertVecRow(Handle, TEXT("items"), /*RowId=*/1, Vec, Err);

// C. K-NN query
TArray<float> Query; Query.SetNum(1536);
// ... fill Query ...
TArray<FVectorSearchResult> Results;
Ok = USQLiteVecBPLibrary::VectorQuery(Handle, TEXT("items"), Query, /*K=*/5, Results, Err);

// Close database
USQLiteVecBPLibrary::CloseDatabase(Handle);
```

---

© 2025 **haozena**. All Rights Reserved.

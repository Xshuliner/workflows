# workflows

## 公共工作流仓库

### 1. Auto Increment Tag

> tag 版本号自增

### 2. Deploy Release

> 部署到仓库的 Release 上

### 3. Deploy Statistics

> 部署统计

### 4. Upload Html

> 上传 HTML

## Dev

```
act --container-architecture linux/amd64 -j increment-tag
act --container-architecture linux/amd64 -j stats
act --container-architecture linux/amd64 -j upload
```

# 云端 3D 摄影在线展厅

这是一个可部署到 GitHub Pages 的静态 3D 摄影展厅。用户把图片拖进网页后，会立即生成可旋转、缩放、点击预览的 3D 展厅。

## 当前能力

- 拖拽或选择多张图片自动布展
- 四种展厅空间：高级沙龙、黑盒美术馆、博物馆长廊、中岛展墙
- 可切换布展密度
- 可摆放不同身高和姿态的观众
- 可点击作品放大预览
- 可直接部署到 GitHub Pages
- 配置 Supabase 后可生成别人能打开的云端分享链接

## 为什么需要 Supabase

GitHub Pages 只能托管静态文件，不能替用户保存上传图片。  
如果希望“用户上传图片后，别人也能打开同一个展厅”，需要一个云端数据库和图片存储。本项目默认预留 Supabase 接口。

## Supabase 配置

1. 新建 Supabase 项目。
2. 新建公开 Storage bucket，名称为 `exhibition-photos`。
3. 新建数据表 `exhibitions`：

```sql
create table exhibitions (
  id uuid primary key,
  title text,
  space_type text,
  density text,
  photos jsonb,
  audience_positions jsonb,
  created_at timestamptz default now()
);
```

4. 在 Supabase 的 SQL Editor 中开启匿名读写策略，适合原型阶段：

```sql
alter table exhibitions enable row level security;

create policy "Public exhibitions read"
on exhibitions for select
using (true);

create policy "Public exhibitions insert"
on exhibitions for insert
with check (true);
```

5. 在 `index.html` 中填写：

```js
const SUPABASE_URL = "你的 Supabase Project URL";
const SUPABASE_ANON_KEY = "你的 Supabase anon public key";
```

6. 给 `exhibition-photos` bucket 添加公开读取和上传策略。原型阶段可以允许匿名上传；正式产品建议增加登录、文件大小限制和审核。

## GitHub Pages 部署

1. 把本文件夹上传到一个 GitHub 仓库。
2. 仓库中保留 `index.html` 在根目录。
3. 打开仓库 Settings -> Pages。
4. Source 选择 `Deploy from a branch`。
5. Branch 选择 `main`，目录选择 `/root`。
6. 保存后等待 GitHub Pages 生成访问地址。

部署完成后，首页就是在线展厅。配置 Supabase 后，用户点击“保存云端展厅”会生成 `?id=...` 分享链接。

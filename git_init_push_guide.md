# Git 项目初始化与推送笔记:smile:

## 项目结构:star:

```b
jupyter_notebook_pro/ ← Git仓库根目录
└── micro_grad/ ← 项目文件夹
	├── micro_gradient.ipynb ← 项目文件
	└── .ipynb_checkpoints/ ← Jupyter自动生成
└── README.md
```

## 1. 进入项目根目录:arrow_down:
```bash
cd /e/software_DEV/pycharm/pycharm_project/jupyter_notebook_pro
```

## 2. 初始化 Git 仓库（默认分支为 main）:arrow_lower_left:
```bash
git init -b main
```
> 若你的 Git 版本不支持 `-b`：  
> 先 `git init`，再 `git branch -M main`。

## 3. 添加所有文件到暂存区:arrow_lower_left:
```bash
git add .
```

## 4. 提交初始版本:arrow_down:
```bash
git commit -m "feat: initial commit with project structure"
```

## 5. 连接到远程仓库:arrow_down:
```bash
git remote add origin https://github.com/aaaa-ae/jupyter_pro.git
```

## 6. 推送到 main 分支:arrow_lower_left:
```bash
git push -u origin main
```

---

### ✅ 快速检查
```bash
git status          # 查看当前状态
git branch          # 确认当前分支是 main
git remote -v       # 确认远程地址已配置为 origin
```

### ⚠️ 常见问题速查
- **`error: src refspec main does not match any`**  
  说明没有任何提交或当前分支不是 `main`：  
  
  ```bash
  git commit -m "init" --allow-empty  # 或先做一次真实提交
  git branch -M main                  # 强制把当前分支命名为 main
  ```


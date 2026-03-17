Cách phổ biến nhất để **tự động bump version + generate changelog từ commit message** là dùng chuẩn **Conventional Commits** cùng với tool như **semantic-release**.

Dưới đây là workflow chuẩn mà rất nhiều project open-source dùng.

---

# 1️⃣ Commit message quyết định version

Semantic-release đọc commit message để quyết định **bump version**.

| Commit             | Version bump            |
| ------------------ | ----------------------- |
| `fix:`             | patch → `1.0.0 → 1.0.1` |
| `feat:`            | minor → `1.0.0 → 1.1.0` |
| `BREAKING CHANGE:` | major → `1.0.0 → 2.0.0` |

Ví dụ commit history:

```bash
feat: add image generation API
fix: resolve GPU memory issue
docs: update readme
```

Release tiếp theo sẽ là:

```
1.1.0
```

---

# 2️⃣ Breaking change

Nếu commit có **breaking change**, version sẽ tăng major.

```bash
feat: redesign API

BREAKING CHANGE: change response format
```

Version:

```
1.0.0 → 2.0.0
```

---

# 3️⃣ Changelog được generate tự động

Semantic-release sẽ tạo changelog kiểu này:

```markdown
# 1.1.0 (2026-03-17)

### Features
- add image generation API

### Bug Fixes
- resolve GPU memory issue
```

Không cần viết changelog bằng tay.

---

# 4️⃣ Setup semantic-release (Node project)

Cài package:

```bash
npm install --save-dev semantic-release
```

Thêm config vào `package.json`:

```json
{
  "release": {
    "branches": ["main"]
  }
}
```

---

# 5️⃣ GitHub Action tự động release

Tạo file:

```
.github/workflows/release.yml
```

```yaml
name: Release

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm install

      - run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Sau mỗi push vào `main`:

1️⃣ semantic-release đọc commit
2️⃣ quyết định version
3️⃣ tạo **GitHub Release**
4️⃣ generate **changelog**

---

# 6️⃣ Ví dụ commit workflow

Dev chỉ cần commit như sau:

```bash
git commit -m "feat: support SDXL turbo model"
git commit -m "fix: handle CUDA OOM error"
git commit -m "docs: update installation guide"
```

Push:

```bash
git push origin main
```

GitHub Action sẽ tự:

```
release v1.1.0
generate changelog
create GitHub tag
```

---

# 7️⃣ Tool để enforce commit format

Để dev không viết sai format:

* commitlint
* husky

Ví dụ commit sai:

```
added new feature
```

commitlint sẽ chặn và yêu cầu:

```
feat: add new feature
```

---

# 8️⃣ Workflow chuẩn của nhiều repo lớn

Flow thường là:

```
branch → commit (conventional) → PR → merge main → CI release
```

Ví dụ commit:

```
feat(api): add image generation endpoint
fix(gpu): prevent cuda memory leak
refactor(model): simplify pipeline loading
```

---

💡 **Tip cho project AI của bạn**

Nếu repo **AI Image Generator (Modal + Diffusers)** của bạn dùng workflow này, bạn sẽ có:

* auto **versioning**
* auto **GitHub release**
* auto **changelog**
* commit history **rất sạch**

---

Nếu bạn muốn, tôi có thể chỉ thêm **một trick mà nhiều dev GitHub không biết**:

👉 cách viết commit để **GitHub tự đóng issue hoặc PR chỉ bằng commit message** (không cần thao tác gì thêm).

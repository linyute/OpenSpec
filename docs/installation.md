# 安裝

## 前置條件

- **Node.js 20.19.0 或更高版本** — 檢查您的版本：`node --version`

## 套件管理員 (Package Managers)

### npm

```bash
npm install -g @fission-ai/openspec@latest
```

### pnpm

```bash
pnpm add -g @fission-ai/openspec@latest
```

### yarn

```bash
yarn global add @fission-ai/openspec@latest
```

### bun

```bash
bun add -g @fission-ai/openspec@latest
```

## Nix

無需安裝直接執行 OpenSpec：

```bash
nix run github:Fission-AI/OpenSpec -- init
```

或安裝到您的個人設定檔 (profile)：

```bash
nix profile install github:Fission-AI/OpenSpec
```

或在 `flake.nix` 中新增到您的開發環境：

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    openspec.url = "github:Fission-AI/OpenSpec";
  };

  outputs = { nixpkgs, openspec, ... }: {
    devShells.x86_64-linux.default = nixpkgs.legacyPackages.x86_64-linux.mkShell {
      buildInputs = [ openspec.packages.x86_64-linux.default ];
    };
  };
}
```

## 驗證安裝

```bash
openspec --version
```

## 後續步驟

安裝後，在您的專案中初始化 OpenSpec：

```bash
cd your-project
openspec init
```

請參閱 [入門指南](getting-started.md) 瞭解完整的導覽說明。

# Mixtile OpenWrt Maintenance

## Branch model

- `openwrt-24.10`: Mixtile maintenance branch for OpenWrt `24.10.x`
- `openwrt-25.12`: primary Mixtile maintenance branch for OpenWrt `25.12.x`
- `main`: fast-forward mirror of `openwrt-25.12`; never develop on it directly

New Mixtile changes land in `openwrt-25.12` first. Backport to
`openwrt-24.10` only after the same change is proven to be compatible.

## Release tags

- `mixtile-v24.10.x-rN`
- `mixtile-v25.12.x-rN`

Examples:

- `mixtile-v24.10.6-r1`
- `mixtile-v25.12.2-r1`

`24.10` remains supported only while upstream OpenWrt supports the series.
Freeze the branch after upstream declares `24.10` EOL.

## Release configs

This repository keeps one Mixtile release config per board and stable series:

- `configs/mixtile-edge2-24.10.config`
- `configs/mixtile-blade3-24.10.config`
- `configs/mixtile-edge2-25.12.config`
- `configs/mixtile-blade3-25.12.config`

Each config builds only `rockchip/armv8` and enables exactly one profile:

- `mixtile_edge2`
- `mixtile_blade3`

## SSH-first validation

Do not push branches or release tags before the SSH validation flow succeeds.

Recommended remote layout:

- `/home/mixtile/work/mixtile-openwrt-24.10-edge2`
- `/home/mixtile/work/mixtile-openwrt-24.10-blade3`
- `/home/mixtile/work/mixtile-openwrt-25.12-edge2`
- `/home/mixtile/work/mixtile-openwrt-25.12-blade3`
- `/home/mixtile/work/releases/24.10/<candidate-tag>/`
- `/home/mixtile/work/releases/25.12/<candidate-tag>/`

Reuse only the historic download cache from
`/home/mixtile/work/mixtile-openwrt-test/dl`. Keep `.config`, `build_dir`,
`staging_dir`, and `bin/` isolated per board directory.

Validation sequence:

1. Check out the target maintenance branch in a clean board-specific git directory.
2. Run `./scripts/feeds update -a && ./scripts/feeds install -a`.
3. Copy the matching `configs/mixtile-<board>-<series>.config` to `.config`.
4. Run `make defconfig`.
5. Run `make download -j12`.
6. Run `make -j12 BUILD_LOG=1`.
7. Confirm `bin/targets/rockchip/armv8/` contains the expected board image,
   `sha256sums`, `profiles.json`, and `*.buildinfo`.

## GitHub publication order

Publish only after SSH validation passes:

1. Push `openwrt-24.10`
2. Push `openwrt-25.12`
3. Fast-forward `main` to `openwrt-25.12`
4. Push maintenance docs and `.github/workflows/mixtile-release.yml`
5. Create `mixtile-v24.10.x-rN` or `mixtile-v25.12.x-rN`

Archive old migration branches with tags before deleting the remote refs.

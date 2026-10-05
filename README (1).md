# FUR-602 专用 bl-mt798x U-Boot 配置

参考机型：qihoo 360T7（同为 MT7981B + MT7531 + 256MB DDR3 + SPI-NAND）。
仓库：https://github.com/hanwckf/bl-mt798x

## 文件放置

把本目录下的文件按原路径拷进 bl-mt798x 仓库根目录：

```
atf-20240117-bacca82a8/configs/mt7981_fur602_defconfig
uboot-mtk-20230718-09eda825/configs/mt7981_fur602_defconfig
uboot-mtk-20230718-09eda825/arch/arm/dts/mt7981-fur-602.dts
```

## 编译

```
sudo apt install gcc-aarch64-linux-gnu build-essential flex bison libssl-dev device-tree-compiler
cd bl-mt798x
SOC=mt7981 BOARD=fur602 ./build.sh
```

产物：

- `output/mt7981_fur602-fip-fixed-parts.bin` —— FIP（uboot.html / mtd write 用）
- BL2 在 `atf-20240117-bacca82a8/build/mt7981/release/bl2.img`（build.sh 默认不拷贝，
  除非 ATF defconfig 加 `_ALL_NO_SEC_BOOT=y`；手动取出来重命名为
  `mt7981_fur602-bl2.bin` 即可）

## 刷入

NAND 机型，ubootmod 布局，bl2 和 fip 必须成对使用（本配置同源编译，直接配套）：

- Web 恢复：bl2 走 `192.168.1.1/bl2.html`，fip 走 `192.168.1.1/uboot.html`
- 系统 SSH：`mtd write /tmp/xxx-bl2.bin bl2`、`mtd write /tmp/xxx-fip.bin fip`，每步后 `sync`

## ⚠️ 刷机前必核对

defconfig 里 mtdparts 写的是（按恩山帖的布局描述）：

```
nmbm0:1024k(bl2),512k(u-boot-env),2048k(factory),512k(trace),2048k(fip),114M(ubi)
```

**必须和 Padavan 内核 dts 的分区表逐项核对**（尤其 trace 是否真的占 512k、
起点对不对）。U-Boot 的 mtdparts 和内核不一致时，fixed-mtdparts 模式下
U-Boot 会按这份表去 ubi 偏移找 kernel，对不上就起不来固件。
核对命令：刷完 U-Boot 后在 U-Boot 串口里 `mtd list` 对照。

## 与 360T7 的差异（已改）

| 项目 | 360T7 | FUR-602 |
|------|-------|---------|
| 绿灯（system） | GPIO 7 | GPIO 8 |
| 红灯（run） | GPIO 3 | GPIO 13 |
| 按钮 | reset=GPIO1 / mesh=GPIO0 | reset=GPIO1 / wps=GPIO0 |
| 设备树名 | mt7981-360t7 | mt7981-fur-602 |
| mtdparts | 带 stock 分区 | ubootmod 布局（同 RAX3000M-NAND） |

未改：eth（gmac0 + mt7531 fixed-link 2500base-x、switch reset GPIO39）、
UART0、SPI-NAND、内存 256MB —— 与 FUR-602 一致。

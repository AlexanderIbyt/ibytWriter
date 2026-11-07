# ibytWriter

ibytWriter writes a boot image (such as `MyProgram.img`, built with the ibyt compiler in `ibyt_bare` mode) to a USB stick, so that a PC can start it without any operating system.

It is written in [ibyt](https://ibyt.tech/) and runs on **Linux** (x86-64). It is a small static program (about 19 KB): no libc, no other dependencies.

> **Warning:** ibytWriter overwrites everything on the drive you choose. Check the number and the model name before you confirm. There is no undo.

## Use

```
sudo ./ibytWriter MyProgram.img
```

1. ibytWriter checks the image (it must end its first sector with the boot signature `55 AA`) and lists the removable USB drives with number, size and model.
2. You choose a drive by number (`q` quits).
3. If the drive is mounted, ibytWriter shows the mount points and asks whether it should unmount them. Answer `y` to continue.
4. ibytWriter asks for a last confirmation: `Write MyProgram.img (… bytes) on /dev/sdb … Continue? (y/n)`.
5. It clears the first and last MiB of the drive (old partition tables, file system leftovers), writes the image, flushes it with `fsync`, reads it back and compares it byte for byte. You see `… bytes written and verified`.
6. Plug the stick into the target PC and choose it in the boot menu. The image has a BIOS boot sector, so set the PC to **Legacy/CSM** boot, not pure UEFI (and turn Secure Boot off).

ibytWriter needs root rights to write to a drive. Without `sudo` it stops at once and says so.

## Safety rules

- Only whole drives, never partitions (`sdb`, not `sdb1`).
- Only removable drives. Fixed disks are refused unless you add `--allow-fixed`.
- Never a drive that is mounted: ibytWriter offers to unmount it, or refuses.
- Never a write-protected drive, and only if the image fits.
- Always a confirmation (`y`), except with `--yes`.
- The result is verified by reading it back.

## More options

For scripts and tests:

```
ibytwriter <image> <drive|file> [--yes] [--allow-fixed] [--raw] [--no-wipe]
ibytwriter --list [--all]
ibytwriter --help
```

| Option | Meaning |
|---|---|
| `<drive>` | a whole drive, e.g. `sdb` or `/dev/sdb` |
| `<file>` | anything else is a file; the image is written to its beginning (no root needed, nothing is cleared) |
| `--yes` | no confirmation, and it never unmounts for you |
| `--allow-fixed` | also allow drives not marked removable (never a mounted one) |
| `--raw` | do not require the boot signature at byte 510 |
| `--no-wipe` | do not clear the first and last MiB before writing |
| `--list` | show the removable drives (`--all`: all real drives) |

## Build

You need the ibyt compiler. Put the `library/` folder of the ibyt project next to the source files and run the compiler in this folder:

```
Main.ibyt  Disk.ibyt  library/
```

The compiler creates the program `MyProgram`; rename it to `ibytWriter`.

## Tested

ibytWriter has automatic tests (about 175 checks with loop devices, fake drives and a mount table in blocks) and was used on a real USB stick to start a Lenovo IdeaPad 110 with an ibyt bare-metal program. Other hardware and BIOS versions may behave differently; please try it with a stick that holds no data first.

## Limits

- Linux only. It reads `/sys/block` and `/proc/mounts` and uses Linux system calls directly.
- x86-64 only.
- It writes raw images. It does not create file systems or copy files.

## License

MIT, see [LICENSE](LICENSE). The license covers ibytWriter. The ibyt compiler and its library are separate and have their own terms.

## Author
(c) Alexander Zima , https://www.ibyt.tech

Alexander Zima, creator of the [ibyt](https://ibyt.tech/) programming language.

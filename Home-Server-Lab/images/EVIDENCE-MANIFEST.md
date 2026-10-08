# Screenshot provenance and evidence manifest

Images are **byte-for-byte copies** of the supplied original PNG files. No cropping, recompression, pixel editing, or embedded PNG metadata changes were performed. Renaming does not alter the embedded image properties. ZIP entry timestamps are not proof of original capture time; original filenames are retained below.

| GitHub filename | Original supplied filename | SHA-256 |
|---|---|---|
| `server-01-proxmox-vm-hardware.png` | `dc01-proxmox-hardware.png` | `c8610cdb0aa6cebe3f626ae3e384bd77fb9df2f89fa8fadc345ba9a7a3b62aa7` |
| `server-02-ad-ds-promotion-options.png` | `ad-ds-review-options.png` | `a43bd99de1e74c1f8b3a1180b0ecaac8b6257e710d1d0b04a9f570a9e591d468` |
| `server-03-ad-domain-validation.png` | `Get-ADDomain.png` | `d728608ef453898045044965cd4bcbfe4d1150fecbafa1e127ba6b5830dd2989` |
| `server-04-ad-forest-dns-validation.png` | `Get-ADForest.png` | `26100edfa1531f2bb5616bae0447c1882d9c14f10c9d82365dbc6b2ca40a85fb` |
| `server-05-virtio-network-driver.png` | `03-virtio-drivers-validated.png` | `04a55f68e34c9cc13c36a08c4799b839742813d52819ca1de45cdeb21ee55335` |
| `client-01-windows-11-deployment.png` | `image(20261008-070046).png` | `ba44578f438fe0121981a6935ec0a6b543b4420da9e07704df807d7ea5b356c7` |
| `client-02-initial-configuration.png` | `image(20261008-070320).png` | `663d2d9aa1cf8366015ae9e8a27d6c2696b3aa5ef8ed109e4d365a3047279dd9` |
| `client-03-windows-installation.png` | `image(20261008-070914).png` | `4f60395a87c257801ddfde49ab586ef3a48632d472d4ff1a35d352d9ab71baaa` |
| `client-04-network-baseline.png` | `image(20261008-071549).png` | `c3d3f5b6332172d8fe3806cb62f512ef0352ecd0b8fcfa38a17230b4203c8ca2` |
| `client-05-dns-diagnostics.png` | `image(20261008-072745).png` | `76297e0d69a53f520679b8589f3d0299efa97e911b1242e2744a65b40ea8f2c0` |
| `client-06-dns-resolution-troubleshooting.png` | `image(20261008-073500).png` | `643b3158055d7626cc2b8e21cf4590983b98d9ac624f8bd0fdd4f2c9f6b35420` |
| `client-07-pre-join-system-properties.png` | `image(20261008-074031).png` | `7080445566d03219fc0eadf70745f447053f89bddeaf97076c4316699106fbbe` |
| `client-08-domain-join-restart-required.png` | `image(20261008-074359).png` | `6c6700298b068ddefbde0d20240e78f809de97f332879e1d4813ba80758391b1` |
| `client-09-cim-verification-error.png` | `image(20261008-075754).png` | `8a6178be2278e37765f2a7b05d628fc71dfdf99e0600c9b984b099da9dc8af6f` |
| `client-10-secure-channel-verified.png` | `image(20261008-080302).png` | `635ca5f74bccf08a1b9a9590bd7aa703a286e28ea7b676858e0a151de28ba3db` |

Some client screenshots are intermediate steps rather than required proof. Review images before publishing, especially browser tabs, account names, IP addresses, and any sensitive details. The AD computer object screenshot has **not** been supplied and is not included.

The screenshot named `client-09-cim-verification-error.png` documents an unresolved CIM/WMI query failure, not a completed fix.

# .claude — ROS 2 / Nav 2 + Gazebo Sim Clean Architecture Template

Bu klasör, ROS 2 (özellikle Nav 2) **ve** Gazebo Sim (gz-sim, ECS C++
tarafı) projelerinde Claude Code'un Clean Architecture ilkelerine bağlı
kalarak çalışmasını sağlayan kalıcı yapılandırmadır. Yeni bir proje
açtığınızda bu template'i `.claude/` olarak kopyalayıp doğrudan
kullanabilirsiniz.

> İki paralel set yan yana çalışır:
> * **ROS 2 / Nav 2** tarafı → colcon, ament, pluginlib, Clean
>   Architecture katmanları, Python+C++.
> * **gz-sim / Gazebo** tarafı → cmake / ninja / bazel, Entity-Component-
>   System, plugin scaffolding.
>
> Komut adları **prefix** ile ayrılır: `/build` (colcon) ⟷ `/gz-build`
> (cmake); `/test` ⟷ `/gz-test`; `ros2-style-reviewer` ⟷
> `gz-style-reviewer`; `clean-arch-architect` ⟷ `ecs-architect`.
> Hangisini istediğinizi bilerek çağırın; ikisi de aktif olarak yüklenir.

## Hızlı bakış

```
.claude/
├── README.md            # bu dosya (Türkçe genel bakış)
├── CLAUDE.md            # Claude'a yüklenen kalıcı proje bağlamı
├── settings.json        # izinler / hook'lar / env değişkenleri
├── commands/            # /slash komutları (terminalden çağrılır)
│   │  ── ROS 2 / Nav 2 tarafı ──
│   ├── build.md             — /build           colcon build sarmalı
│   ├── test.md              — /test            colcon test + result
│   ├── lint.md              — /lint            ament_* + pre-commit
│   ├── new-package.md       — /new-package     ROS 2 paketi iskelesi
│   ├── new-node.md          — /new-node        Node iskelesi
│   ├── new-launch.md        — /new-launch      Launch dosyası iskelesi
│   ├── new-nav2-plugin.md   — /new-nav2-plugin Nav 2 plugin iskelesi
│   ├── changelog.md         — /changelog       Branch için CHANGELOG.rst
│   ├── ros2.md              —                  ROS 2 referans kart
│   ├── nav2.md              —                  Nav 2 referans kart
│   │  ── gz-sim / Gazebo tarafı ──
│   ├── gz-build.md          — /gz-build        cmake + ninja gz-sim build
│   ├── gz-test.md           — /gz-test         ctest / integration tests
│   ├── gz-lint.md           — /gz-lint         pre-commit (gz-sim)
│   ├── gz-new-component.md  — /gz-new-component ECS component scaffold
│   ├── gz-new-system.md     — /gz-new-system   ECS system plugin scaffold
│   └── gz-changelog.md      — /gz-changelog    gz-sim Changelog.md blok
├── skills/              # Claude'un başvuracağı uzun-form reçeteler
│   │  ── ROS 2 ──
│   ├── ros2_node_creation/        — Clean-arch uyumlu node
│   ├── ros2_lifecycle/            — Managed lifecycle node
│   ├── ros2_messaging/            — Pub / Sub kalıpları
│   ├── ros2_service_action/       — Service & Action sarmalama
│   ├── ros2_launch_config/        — Modüler launch
│   ├── ros2_transforms/           — TF2 yönetimi
│   ├── ros2_diagnostics/          — Sağlık izleme
│   ├── ros2_bag/                  — Rosbag kaydı/oynatımı
│   ├── ros2_testing/              — GTest / pytest / launch_testing
│   ├── new_ros2_package/          — Paket scaffold reçetesi
│   │  ── Nav 2 ──
│   ├── new_nav2_plugin/           — Nav 2 plugin scaffold reçetesi
│   ├── nav2_core_interfaces/      — Plugin temel sınıfları
│   ├── nav2_controllers/          — DWB / MPPI / RPP …
│   ├── nav2_planners/             — NavFn / SMAC / Theta* …
│   ├── nav2_behavior_tree/        — BT düğümleri
│   ├── nav2_costmap/              — Costmap katmanları
│   ├── nav2_localization/         — AMCL / Map Server
│   ├── nav2_servers/              — Lifecycle / Velocity / Collision …
│   │  ── gz-sim / Gazebo ──
│   ├── gz-build/                  — cmake / ninja / colcon / bazel reçeteleri
│   ├── gz-test/                   — ctest filtreleri, gtest, headless
│   ├── gz-ecs-overview/           — Entity-Component-System mimari
│   ├── new-component/             — Component header taslakları
│   └── new-system/                — System plugin tam iskelesi
├── rules/               # Projede uyulması gereken kurallar
│   ├── clean_architecture.md      — Katman sınırları
│   ├── ros2_general.md            — Genel ROS 2 prensipleri
│   ├── ros2_nodes.md              — Node geliştirme kuralları
│   ├── ros2_communication.md      — Topic / QoS / interface kuralları
│   ├── testing.md                 — Test stratejisi
│   ├── nav2_architecture.md       — Nav 2 sistem mimarisi
│   ├── nav2_parameters.md         — Nav 2 parametre referansı
│   ├── nav2_msgs_reference.md     — Nav 2 msg/srv/action listesi
│   └── robot_specific.md          — Robot-özel ayarlar (override edin)
└── agents/              # Özel sub-agent tanımları
    │  ── ROS 2 / Nav 2 ──
    ├── ros2-style-reviewer.md     — ROS 2 / Clean-arch PR review uzmanı
    ├── clean-arch-architect.md    — Katman / bağımlılık tasarım danışmanı
    │  ── gz-sim / Gazebo ──
    ├── gz-style-reviewer.md       — gz-sim stil / PR review uzmanı
    └── ecs-architect.md           — ECS tasarım danışmanı
```

## Nasıl kullanılır

### Slash komutlar
Terminalden doğrudan çağrılır:

```
# ROS 2 / Nav 2
/build                    # tüm workspace (colcon)
/build my_pkg             # sadece bir paket
/test                     # colcon test + result
/lint --all               # tüm dosyalar
/new-package my_pkg python
/new-node my_pkg my_node lifecycle cpp
/new-nav2-plugin controller MyController
/changelog

# gz-sim / Gazebo
/gz-build                 # cmake -B build && ninja
/gz-test                  # ctest --test-dir build
/gz-lint                  # pre-commit (gz-sim)
/gz-new-component Foo
/gz-new-system fancy_drive FancyDrive
/gz-changelog
```

### Skills
Claude bir göreve geldiğinde otomatik olarak ilgili `SKILL.md`'ye
başvurur. Bir skill'i açıkça çalıştırmak için doğrudan dosya yolunu
verebilirsiniz:

```
.claude/skills/ros2_lifecycle/SKILL.md'deki tüm adımları uygula
```

### Sub-agents
`Agent` aracıyla çağrılır:

ROS 2 / Nav 2 tarafı:
* `subagent_type: "ros2-style-reviewer"` — diff'in clean-arch ve
  ROS 2 kurallarına uyumunu sıkı şekilde denetler, line-anchored
  geri bildirim verir.
* `subagent_type: "clean-arch-architect"` — yeni bir özelliğin
  hangi katmana hangi şekilde yerleşmesi gerektiği üzerine seçenekli
  öneri üretir.

gz-sim / Gazebo tarafı:
* `subagent_type: "gz-style-reviewer"` — gz-sim stil, plugin
  registration, CMake/Bazel uyumunu denetler.
* `subagent_type: "ecs-architect"` — yeni state/davranışın hangi
  Component'a veya hangi System fazına ait olduğu üzerine seçenekli
  öneri üretir.

### Kişisel geçersiz kılma

Repo'ya commit etmek istemediğiniz kişisel izin/env ayarları için
`.claude/settings.local.json` dosyasını oluşturun (otomatik olarak
.gitignore'a eklendi).

## Yeni içerik eklerken

* Yeni bir skill: `.claude/skills/<kebab_or_snake>/SKILL.md` —
  frontmatter'da `name` ve `description` **zorunlu**.
* Yeni bir slash komut: `.claude/commands/<name>.md` —
  frontmatter'da `description`, isteğe bağlı `argument-hint` ve
  `allowed-tools`. Gövdede `$ARGUMENTS` ile parametre alınır.
* Yeni bir sub-agent: `.claude/agents/<name>.md` — `name`,
  `description`, opsiyonel `tools` ve `model`.
* Her eklemede bu README'deki dizini güncelleyin.

## Felsefe

1. **Bağımlılığı tersine çevir.** `domain/` ROS 2'yi bilmez; ROS 2
   bağlamı `infrastructure/` katmanında durur.
2. **Önce reçete, sonra kod.** Komutlar küçük yardımcılardır;
   skills ve rules sayesinde Claude'un ürettiği kod tutarlıdır.
3. **Açık komutlar > sihirli otomatik davranış.** Otomatik
   davranışlar `settings.json` hook'larıyla ifade edilir; gizli
   yan-etki üretmeyiz.
4. **Tek kaynak.** Her bilgi tek bir yerde durur; dublikasyon yerine
   `[[rule_dosyasi]]` formatında bağlantı kurun.

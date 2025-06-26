# Welcome to `MkDocs`

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

## Project layout
Для публикации на GitHub Pages структуру `mkdocs` лучше поместить внутрь подпапки:

    mkdocs/
        mkdocs.yml      # The configuration file
        docs/
            index.md    # The homepage.
                ...     # Other markdown pages

## Публикация в Github Pages

1. Создать репозиторий вида `[имя_пользователя].github.io` (однократно, один для пользователя - корень web-сайта):
![GitHub Pages Repositiry](../assets/img/create-repository-name-pages.webp "GitHub Pages Repositiry"){ width="500" }

    Любой из файлов `index.html`, `index.md` или `README.md` будет отображаться GitHub Pages, как стартовая страница web-сайта `https://[имя_пользователя].github.io`

1. В любом другом репозитории этого же пользователя можно создать папку `docs`, в которой можно разместить документацию, которая будет выводиться web-сервером по адресу `https://[имя_пользователя].github.io/[имя_репозитория]`

1. Для этого в Настройках (`Settings`) репозитория необходимо перейти в раздел `Pages`, настроить источник (Source) `Deploy from a branch`, задать исходную ветку, при обновлении которой будет опубликовано содержимое документации, и папку (`/` или `/docs`)

1. Также можно задать (при наличии) свой домен для web-адреса (вместо `[имя_пользователя].github.io`) и протокол (`http` или `https`)

1. Если в качестве источника (Source) выбрать `GitHub Actions`, то можно использовать различные способы автоматизации до публикации. Например, запускать контейнер, в котором будет собираться проект одностраничного web-сайта и после этого публиковаться в GitHub Pages.
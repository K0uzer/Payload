# Payload
- Информация бралась из:
 1. https://github.com/K0uzer/production-project-95625fa1c42f772092f37824ef52a4be1fa467f9

# SCSS/CSS/CSS-IN-JS


# Animations
## https://cssloaders.github.io/


# Layout


# JS


# TS
- https://astexplorer.net/ - визуализатора абстрактных синтаксических деревьев (AST), созданных различными парсерами. 

# React + ecosystem
- https://use-gesture.netlify.app/docs/gestures/ - drag-and-drop 
- https://www.npmjs.com/package/react-device-detect - помогает узнать размер экрана 
- https://floating-ui.com/ - для работы с popover, popup, их скролл и т.п.

# Lint
- https://www.npmjs.com/package/eslint-plugin-unused-imports - не используемые импорты
- https://www.npmjs.com/package/eslint-plugin-import - работа с импортами

# Tests
- https://jestjs.io/docs/tutorial-react 
- https://testing-library.com/docs/queries/about/
- https://loki.js.org/getting-started.html
- https://www.cypress.io/ - e2e тесты ( отлавливаем ошибки уже на реальных данных, тоесть проверка глобальной работоспособности проекта в связке с бэком. НО! e2e тесты пишутся только для чего-то глобального т.к. они очень дорогие! поэтому сначало все покрой unit и скрин тестами, а только потом e2e! )
- https://www.npmjs.com/package/husky - husky ( запуск пре-коммитов ( тесты, линтеры и т.д. ) )

# Safety


# Asymptotic complexity
- https://big-o-calculator.vercel.app/
- https://www.bigocalc.com/

# UI-kit
- https://headlessui.com/ - headless-ui
- https://react-spectrum.adobe.com/react-aria/index.html - hooks, functions
- https://react-spring.dev/docs
# Bundle
- https://bundlephobia.com/ - узнай сколько займет библиотека в бандле и устройство
- Хочешь ускорить сборку? Разбей на несколько процессов, так же ts и tsx должны быть разбельными в процессах! vite?
# Code helpers

## РАБОТА С НОДАМИ. Удалить старую Фичу из кода (чекай https://astexplorer.net/) - проходится по всему проекту и удаляет код, который ты выбрал (on или off)

    import { JsxAttribute, Node, Project, SyntaxKind } from 'ts-morph';
    
    const removedFeatureName = process.argv[2]; // example isArticleEnabled
    const featureState = process.argv[3]; // example off\on
    
    const toggleFunctionName = 'toggleFeatures';
    const toggleComponentName = 'ToggleFeatures';
    
    if (!removedFeatureName) {
        throw new Error('Укажите название фича-флага');
    }
    
    if (!featureState) {
        throw new Error('Укажите состояние фичи (on или off)');
    }
    
    if (featureState !== 'on' && featureState !== 'off') {
        throw new Error('Некорректное значение состояния фичи (on или off)');
    }
    
    const project = new Project({});
    
    project.addSourceFilesAtPaths('src/**/ArticleDetailsPage.ts');
    project.addSourceFilesAtPaths('src/**/ArticleDetailsPage.tsx');
    
    const files = project.getSourceFiles();
    
    function isToggleFunction(node: Node) {
        let isToggleFeatures = false;
    
        node.forEachChild((child) => {
            if (
                child.isKind(SyntaxKind.Identifier) &&
                child.getText() === toggleFunctionName
            ) {
                isToggleFeatures = true;
            }
        });
    
        return isToggleFeatures;
    }
    
    function isToggleComponent(node: Node) {
        const identifier = node.getFirstDescendantByKind(SyntaxKind.Identifier);
    
        return identifier?.getText() === toggleComponentName;
    }
    
    const replaceToggleFunction = (node: Node) => {
        const objectOptions = node.getFirstDescendantByKind(
            SyntaxKind.ObjectLiteralExpression,
        );
    
        if (!objectOptions) return;
    
        const offFunctionProperty = objectOptions.getProperty('off');
        const onFunctionProperty = objectOptions.getProperty('on');
    
        const featureNameProperty = objectOptions.getProperty('name');
    
        const onFunction = onFunctionProperty?.getFirstDescendantByKind(
            SyntaxKind.ArrowFunction,
        );
        const offFunction = offFunctionProperty?.getFirstDescendantByKind(
            SyntaxKind.ArrowFunction,
        );
        const featureName = featureNameProperty
            ?.getFirstDescendantByKind(SyntaxKind.StringLiteral)
            ?.getText()
            .slice(1, -1);
    
        if (featureName !== removedFeatureName) return;
    
        if (featureState === 'on') {
            node.replaceWithText(onFunction?.getBody().getText() ?? '');
        }
    
        if (featureState === 'off') {
            node.replaceWithText(offFunction?.getBody().getText() ?? '');
        }
    };
    
    const getAttributeNodeByName = (
        jsxAttributes: JsxAttribute[],
        name: string,
    ) => {
        return jsxAttributes.find((node) => node.getName() === name);
    };
    
    const getReplacedComponent = (attribute?: JsxAttribute) => {
        const value = attribute
            ?.getFirstDescendantByKind(SyntaxKind.JsxExpression)
            ?.getExpression()
            ?.getText();
    
        if (value?.startsWith('(')) {
            return value.slice(1, -1);
        }
    
        return value;
    };
    
    const replaceComponent = (node: Node) => {
        const attributes = node.getDescendantsOfKind(SyntaxKind.JsxAttribute);
    
        const onAttribute = getAttributeNodeByName(attributes, 'on');
        const offAttribute = getAttributeNodeByName(attributes, 'off');
    
        const featureNameAttribute = getAttributeNodeByName(attributes, 'feature');
        const featureName = featureNameAttribute
            ?.getFirstDescendantByKind(SyntaxKind.StringLiteral)
            ?.getText()
            ?.slice(1, -1);
    
        if (featureName !== removedFeatureName) return;
    
        const offValue = getReplacedComponent(offAttribute);
        const onValue = getReplacedComponent(onAttribute);
    
        if (featureState === 'on' && onValue) {
            node.replaceWithText(onValue);
        }
    
        if (featureState === 'off' && offValue) {
            node.replaceWithText(offValue);
        }
    };
    
    files.forEach((sourceFile) => {
        sourceFile.forEachDescendant((node) => {
            if (node.isKind(SyntaxKind.CallExpression) && isToggleFunction(node)) {
                replaceToggleFunction(node);
            }
    
            if (
                node.isKind(SyntaxKind.JsxSelfClosingElement) &&
                isToggleComponent(node)
            ) {
                replaceComponent(node);
            }
        });
    });
    
    project.save();


## NGINX.CONFIG

    server {
            root /var/www/production_project/html;
    
            index index.html index.htm index.nginx-debian.html;
    
            server_name productionapputv.ru www.productionapputv.ru;
    
            location  ~ ^/api/(.*)$ {
                proxy_pass   $scheme://80.93.190.163:8443/$1$is_args$args;
                proxy_redirect     off;
                proxy_set_header   Host             $host;
                proxy_set_header   X-Real-IP        $remote_addr;
                proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
            }
    
            location / {
                    try_files $uri $uri/ /index.html;
            }
    
    
        listen [::]:443 ssl ipv6only=on; # managed by Certbot
        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/productionapputv.ru/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/productionapputv.ru/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
    
    
    }
    server {
        if ($host = www.productionapputv.ru) {
            return 301 https://$host$request_uri;
        } # managed by Certbot
    
    
        if ($host = productionapputv.ru) {
            return 301 https://$host$request_uri;
        } # managed by Certbot
    
    
            listen 80;
            listen [::]:80;
    
            server_name productionapputv.ru www.productionapputv.ru;
        return 404; # managed by Certbot
    
    }

## Пример BASH-SCRIPT

    cd ~/production-project
    npm run build:prod
    
    rm -rf ~/../var/www/production_project/html
    mv ~/production-project/build ~/../var/www/production_project/html

## Пример Фича-флага для проекта

ToggleFeatures.tsx

    import { ReactElement } from 'react';
    import { FeatureFlags } from '@/shared/types/featureFlags';
    import { getFeatureFlag } from '../setGetFeatures';
    
    interface ToggleFeaturesProps {
        feature: keyof FeatureFlags;
        on: ReactElement;
        off: ReactElement;
    }
    
    export const ToggleFeatures = (props: ToggleFeaturesProps) => {
        const { on, off, feature } = props;
    
        if (getFeatureFlag(feature)) {
            return on;
        }
    
        return off;
    };

Геттеры и сеттеры для фича-флага - setGetFuatures.ts

    import { FeatureFlags } from '@/shared/types/featureFlags';
    
    // ФИЧИ НЕ МЕНЯЮТСЯ В ХОДЕ СЕССИИ, ИХ НЕОБЯЗАТЕЛЬНО ДЕЛАТЬ РЕАКТИВНЫМИ!
    let featureFlags: FeatureFlags;
    
    export function setFeatureFlags(newFeatureFlags?: FeatureFlags) {
        if (newFeatureFlags) {
            featureFlags = newFeatureFlags;
        }
    }
    
    export function getFeatureFlag(flag: keyof FeatureFlags) {
        return featureFlags[flag];
    }

Переключатель с дженериком - toggleFeatures.ts

    import { FeatureFlags } from '@/shared/types/featureFlags';
    import { getFeatureFlag } from './setGetFeatures';
    
    interface ToggleFeaturesOptions<T> {
        name: keyof FeatureFlags;
        on: () => T;
        off: () => T;
    }
    
    export function toggleFeatures<T>({
        off,
        on,
        name,
    }: ToggleFeaturesOptions<T>): T {
        if (getFeatureFlag(name)) {
            return on();
        }
    
        return off();
    }



## Dev Tools

Mock Service Worker (MSW)
Зачем: Мокает API прямо в браузере, убирая зависимость от бэкенда.

    Ссылка: https://mswjs.io/

Пример в src/mocks/browser.ts:

    import { setupWorker } from 'msw';
    import { handlers } from './handlers';
    export const worker = setupWorker(...handlers);

## Отлавливание лишних ререндеры React-компонентов

    Ссылка: https://github.com/welldone-software/why-did-you-render

Как подключить:

    npm install @welldone-software/why-did-you-render

Вход в index.tsx:

    if (process.env.NODE_ENV === 'development') {
      const whyDidYouRender = require('@welldone-software/why-did-you-render');
      whyDidYouRender(React, { trackAllPureComponents: true });
    }

## Commitlint + cz-cli - Единый формат коммитов

Ссылка: https://commitlint.js.org/

    npm install --save-dev @commitlint/config-conventional @commitlint/cli


## CLI-генератор FSD-срезов

Зачем: Автоматически создаёт структуру по Feature-Sliced Design.
Как подключить: Можно сделать plopfile.js и использовать Plop.js.

    npm install --save-dev plop

## Knip — анализ неиспользуемого кода

Зачем: Находит неиспользуемые импорты, файлы и зависимости.
Ссылка: https://github.com/webpro/knip

    npx knip


## CI/CD и автоматизация

1. Параллельный запуск тестов и линтеров
Зачем: Ускоряет CI, так как задачи выполняются одновременно.
Как подключить: Добавить несколько job в GitHub Actions с зависимостями между ними.

### Простой пример CI

    Вот простой пример CI-конфигурации для GitHub Actions, которая будет автоматически запускать тесты репозитория при каждом пуше и pull request.
    
    .github/workflows/tests.yml
    
    yaml
    Копировать
    Редактировать
    name: Run Tests
    
    on:
      push:
        branches: [ main ]
      pull_request:
        branches: [ main ]
    
    jobs:
      test:
        runs-on: ubuntu-latest
    
        steps:
          - name: Checkout repository
            uses: actions/checkout@v4
    
          - name: Set up Node.js
            uses: actions/setup-node@v4
            with:
              node-version: 20
              cache: npm
    
          - name: Install dependencies
            run: npm ci
    
          - name: Run tests
            run: npm test
    🔹 Как это работает:
    
    on — указывает, когда запускать тесты (в данном случае при пушах и PR в main).
    
    jobs.test — отдельная задача для тестирования.
    
    runs-on — выбираем ОС, на которой будет выполняться.
    
    steps — шаги:
    
    actions/checkout загружает код репозитория.
    
    actions/setup-node ставит нужную версию Node.js и кэширует зависимости.
    
    npm ci устанавливает зависимости из package-lock.json.
    
    npm test запускает тесты.

2. Кэширование зависимостей и сборки
Зачем: Уменьшает время сборки на CI.
Как подключить: Использовать actions/cache для node_modules и .vite / .webpack-cache.

          - uses: actions/cache@v4
            with:
              path: |
                ~/.npm
                .vite
              key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

3. Автоматический деплой Storybook и приложения
Зачем: Чтобы всегда была доступна актуальная сборка UI и стейджинг приложения.
Как подключить: Для Storybook — Chromatic или деплой на GitHub Pages.

- name: Deploy Storybook
  run: npm run storybook:build && npx gh-pages -d storybook-static


## Cypress.config.js - настройка E2E теста

    import { defineConfig } from 'cypress';
    
    export default defineConfig({
        e2e: {
            setupNodeEvents(on, config) {
                // implement node event listeners here
            },
            baseUrl: 'http://localhost:3000/',
        },
    });

### Тест компонента через Cypress

#### Пример теста роутера: e2e > common > routing.cy.ts

    import { selectByTestId } from '../../helpers/selectByTestId';
    
    describe('Роутинг', () => {
        describe('Пользователь НЕ авторизован', () => {
            it('Переход на главную страницу', () => {
                cy.visit('/');
                cy.get(selectByTestId('MainPage')).should('exist');
            });
            it('Переход открывает страницу профиля', () => {
                cy.visit('/profile/1');
                cy.get(selectByTestId('MainPage')).should('exist');
            });
            it('Переход открывает несуществующий маршрут ', () => {
                cy.visit('/fasfasfasf');
                cy.get(selectByTestId('NotFoundPage')).should('exist');
            });
        });
        describe('Пользователь авторизован', () => {
            beforeEach(() => {
                cy.login();
            });
            it('Переход открывает страницу профиля', () => {
                cy.visit('/profile/1');
                cy.get(selectByTestId('ProfilePage')).should('exist');
            });
    
            it('Переход открывает страницу со списком статей', () => {
                cy.visit('/articles');
                cy.get(selectByTestId('ArticlesPage')).should('exist');
            });
        });
    });

 #### support > commands > login.ts

    import { USER_LOCALSTORAGE_KEY } from '../../../src/shared/const/localstorage';
    
    export const login = (username: string = 'testuser', password: string = '123') => {
        cy.request({
            method: 'POST',
            url: 'http://localhost:8000/login',
            body: {
                username,
                password,
            },
        }).then(({ body }) => {
            window.localStorage.setItem(USER_LOCALSTORAGE_KEY, JSON.stringify(body));
        });
    };

#### command.ts

    import { login } from './commands/login';
    
    Cypress.Commands.add('login', login);
    
    declare global {
      namespace Cypress {
        interface Chainable {
          login(email?: string, password?: string): Chainable<void>
        }
      }
    }
    
    export {};


## Hysky - пре-коммит ( проверяем линтером наш код перед отправкой данных на реп )

    #!/bin/sh
    . "$(dirname "$0")/_/husky.sh"
    
    npx lint-staged

## Структура директорий c FSD: 

    App
     ^
    Pages
     ^
    Widgets
     ^
    Features
     ^
    Entities
     ^
    Shared

## Storybook: 
- https://www.npmjs.com/package/eslint-plugin-import - само приложение
- [npm i storybook-addon-themes](https://www.npmjs.com/package/storybook-addon-themes) - смена темы в Storybook

### Кнопка:

    import React from 'react';
    import { ComponentStory, ComponentMeta } from '@storybook/react';
    
    import { ThemeDecorator } from 'shared/config/storybook/ThemeDecorator/ThemeDecorator';
    import { Theme } from 'app/providers/ThemeProvider';
    import { Button, ButtonSize, ButtonTheme } from './Button';
    
    export default {
        title: 'shared/Button',
        component: Button,
        argTypes: {
            backgroundColor: { control: 'color' },
        },
    } as ComponentMeta<typeof Button>;
    
    const Template: ComponentStory<typeof Button> = (args) => <Button {...args} />;
    
    export const Primary = Template.bind({});
    Primary.args = {
        children: 'Text',
    };
    
    export const Clear = Template.bind({});
    Clear.args = {
        children: 'Text',
        theme: ButtonTheme.CLEAR,
    };

### Скелетон:

    import React from 'react';
    import { ComponentMeta, ComponentStory } from '@storybook/react';
    
    import { ThemeDecorator } from 'shared/config/storybook/ThemeDecorator/ThemeDecorator';
    import { Theme } from 'app/providers/ThemeProvider';
    import { Skeleton } from './Skeleton';
    
    export default {
        title: 'shared/Skeleton',
        component: Skeleton,
        argTypes: {
            backgroundColor: { control: 'color' },
        },
    } as ComponentMeta<typeof Skeleton>;
    
    const Template: ComponentStory<typeof Skeleton> = (args) => <Skeleton {...args} />;
    
    export const Normal = Template.bind({});
    Normal.args = {
        width: '100%',
        height: 200,
    };
    
    export const Circle = Template.bind({});
    Circle.args = {
        border: '50%',
        width: 100,
        height: 100,
    };
    
    export const NormalDark = Template.bind({});
    NormalDark.args = {
        width: '100%',
        height: 200,
    };
    NormalDark.decorators = [ThemeDecorator(Theme.DARK)];
    export const CircleDark = Template.bind({});
    CircleDark.args = {
        border: '50%',
        width: 100,
        height: 100,
    };
    CircleDark.decorators = [ThemeDecorator(Theme.DARK)];

## Jests тесты: 

 ### Конфиг JEST:
    import path from 'path';
    
    export default {
        globals: {
            __IS_DEV__: true,
            __API__: '',
            __PROJECT__: 'jest',
        },
        clearMocks: true,
        testEnvironment: 'jsdom',
        coveragePathIgnorePatterns: [
            '\\\\node_modules\\\\',
        ],
        moduleFileExtensions: [
            'js',
            'jsx',
            'ts',
            'tsx',
            'json',
            'node',
        ],
        moduleDirectories: [
            'node_modules',
        ],
        modulePaths: [
            '<rootDir>src',
        ],
        testMatch: [
            // Обнаружил разницу между МАК ОС и ВИНДОУС!!!
            '<rootDir>src/**/*(*.)@(spec|test).[tj]s?(x)',
        ],
        rootDir: '../../',
        setupFilesAfterEnv: ['<rootDir>config/jest/setupTests.ts'],
        moduleNameMapper: {
            '\\.s?css$': 'identity-obj-proxy',
            '\\.svg': path.resolve(__dirname, 'jestEmptyComponent.tsx'),
        }
    }


### Кнопка:

    import { render, screen } from '@testing-library/react';
    import { Button, ButtonTheme } from './Button';
    
    describe('Button', () => {
        test('Test render', () => {
            render(<Button>TEST</Button>);
            expect(screen.getByText('TEST')).toBeInTheDocument();
        });
    
        test('Test clear theme', () => {
            render(<Button theme={ButtonTheme.CLEAR}>TEST</Button>);
            expect(screen.getByText('TEST')).toHaveClass('clear');
            screen.debug();
        });
    });

### Сайдбар:

    import { fireEvent, screen } from '@testing-library/react';
    import { componentRender } from 'shared/lib/tests/componentRender/componentRender';
    import { Sidebar } from '../Sidebar/Sidebar';
    
    describe('Sidebar', () => {
        test('with only first param', () => {
            componentRender(<Sidebar />);
            expect(screen.getByTestId('sidebar')).toBeInTheDocument();
        });
    
        test('test toggle', () => {
            componentRender(<Sidebar />);
            const toggleBtn = screen.getByTestId('sidebar-toggle');
            expect(screen.getByTestId('sidebar')).toBeInTheDocument();
            fireEvent.click(toggleBtn);
            expect(screen.getByTestId('sidebar')).toHaveClass('collapsed');
        });
    });

## Ассинхронная подгрузка:
### Статьи:
### ActiclePage.async.tsx:

    import { lazy } from 'react';
    
    export const ArticlesPageAsync = lazy(
        () => import('./ArticlesPage'),
    );
    
### ActiclePage.tsx:

    import { classNames } from 'shared/lib/classNames/classNames';
    import { useTranslation } from 'react-i18next';
    import { memo, useCallback } from 'react';
    import { ArticleList } from 'entities/Article';
    import { DynamicModuleLoader, ReducersList } from   'shared/lib/components/DynamicModuleLoader/DynamicModuleLoader';
    import { useInitialEffect } from 'shared/lib/hooks/useInitialEffect/useInitialEffect';
    import { useSelector } from 'react-redux';
    import { useAppDispatch } from 'shared/lib/hooks/useAppDispatch/useAppDispatch';
    import { Page } from 'widgets/Page/Page';
    import { useSearchParams } from 'react-router-dom';
    import { ArticlesPageFilters } from '../ArticlesPageFilters/ArticlesPageFilters';
    import { fetchNextArticlesPage } from '../../model/services/fetchNextArticlesPage/fetchNextArticlesPage';
    import { initArticlesPage } from '../../model/services/initArticlesPage/initArticlesPage';
    import { articlesPageReducer, getArticles } from '../../model/slices/articlesPageSlice';
    import cls from './ArticlesPage.module.scss';
    import {
        getArticlesPageError,
        getArticlesPageIsLoading,
        getArticlesPageView,
    } from '../../model/selectors/articlesPageSelectors';
    
    interface ArticlesPageProps {
        className?: string;
    }
    
    const reducers: ReducersList = {
        articlesPage: articlesPageReducer,
    };
    
    const ArticlesPage = (props: ArticlesPageProps) => {
        const { className } = props;
        const { t } = useTranslation();
        const dispatch = useAppDispatch();
        const articles = useSelector(getArticles.selectAll);
        const isLoading = useSelector(getArticlesPageIsLoading);
        const view = useSelector(getArticlesPageView);
        const error = useSelector(getArticlesPageError);
        const [searchParams] = useSearchParams();
    
        const onLoadNextPart = useCallback(() => {
            dispatch(fetchNextArticlesPage());
        }, [dispatch]);
    
        useInitialEffect(() => {
            dispatch(initArticlesPage(searchParams));
        });
    
        return (
            <DynamicModuleLoader reducers={reducers} removeAfterUnmount={false}>
                <Page
                    onScrollEnd={onLoadNextPart}
                    className={classNames(cls.ArticlesPage, {}, [className])}
                >
                    <ArticlesPageFilters />
                    <ArticleList
                        isLoading={isLoading}
                        view={view}
                        articles={articles}
                        className={cls.list}
                    />
                </Page>
            </DynamicModuleLoader>
        );
    };
    
    export default memo(ArticlesPage);

## RTK работа с Store:
### Струкруга директирии в FSD:

    ArticlePage > model > selectors
                          services
                          slices
                          types

 ### Services > Selectors > articlesPageSelectors.ts:
 
    import { StateSchema } from 'app/providers/StoreProvider';
    import { ArticleSortField, ArticleType, ArticleView } from 'entities/Article';
    
    export const getArticlesPageIsLoading = (state: StateSchema) => state.articlesPage?.isLoading || false;
    export const getArticlesPageError = (state: StateSchema) => state.articlesPage?.error;
    export const getArticlesPageView = (state: StateSchema) => state.articlesPage?.view || ArticleView.SMALL;
    export const getArticlesPageNum = (state: StateSchema) => state.articlesPage?.page || 1;
    export const getArticlesPageLimit = (state: StateSchema) => state.articlesPage?.limit || 9;
    export const getArticlesPageHasMore = (state: StateSchema) => state.articlesPage?.hasMore;
    export const getArticlesPageInited = (state: StateSchema) => state.articlesPage?._inited;
    export const getArticlesPageOrder = (state: StateSchema) => state.articlesPage?.order ?? 'asc';
    export const getArticlesPageSort = (state: StateSchema) => state.articlesPage?.sort ?? ArticleSortField.CREATED;
    export const getArticlesPageSearch = (state: StateSchema) => state.articlesPage?.search ?? '';
    export const getArticlesPageType = (state: StateSchema) => state.articlesPage?.type ?? ArticleType.ALL;

 ### Services > fetchArticlesList > fetchArticlesList.ts:
 
    import { createAsyncThunk } from '@reduxjs/toolkit';
    import { ThunkConfig } from 'app/providers/StoreProvider';
    import { Article, ArticleType } from 'entities/Article';
    import { addQueryParams } from 'shared/lib/url/addQueryParams/addQueryParams';
    import {
        getArticlesPageLimit,
        getArticlesPageNum,
        getArticlesPageOrder,
        getArticlesPageSearch,
        getArticlesPageSort,
        getArticlesPageType,
    } from '../../selectors/articlesPageSelectors';
    
    interface FetchArticlesListProps {
        replace?: boolean;
    }
    
    export const fetchArticlesList = createAsyncThunk<
        Article[],
        FetchArticlesListProps,
        ThunkConfig<string>
        >(
            'articlesPage/fetchArticlesList',
            async (props, thunkApi) => {
                const { extra, rejectWithValue, getState } = thunkApi;
                const limit = getArticlesPageLimit(getState());
                const sort = getArticlesPageSort(getState());
                const order = getArticlesPageOrder(getState());
                const search = getArticlesPageSearch(getState());
                const page = getArticlesPageNum(getState());
                const type = getArticlesPageType(getState());
    
                try {
                    addQueryParams({
                        sort, order, search, type,
                    });
                    const response = await extra.api.get<Article[]>('/articles', {
                        params: {
                            _expand: 'user',
                            _limit: limit,
                            _page: page,
                            _sort: sort,
                            _order: order,
                            q: search,
                            type: type === ArticleType.ALL ? undefined : type,
                        },
                    });
    
                    if (!response.data) {
                        throw new Error();
                    }
    
                    return response.data;
                } catch (e) {
                    return rejectWithValue('error');
                }
            },
        );

 ### Services > fetchNextArticlesPage > fetchNextArticlesPage.ts:
 
    import { createAsyncThunk } from '@reduxjs/toolkit';
    import { ThunkConfig } from 'app/providers/StoreProvider';
    import {
        getArticlesPageHasMore,
        getArticlesPageIsLoading,
        getArticlesPageNum,
    } from '../../selectors/articlesPageSelectors';
    import { articlesPageActions } from '../../slices/articlesPageSlice';
    import { fetchArticlesList } from '../fetchArticlesList/fetchArticlesList';
    
    export const fetchNextArticlesPage = createAsyncThunk<
        void,
        void,
        ThunkConfig<string>
        >(
            'articlesPage/fetchNextArticlesPage',
            async (_, thunkApi) => {
                const { getState, dispatch } = thunkApi;
                const hasMore = getArticlesPageHasMore(getState());
                const page = getArticlesPageNum(getState());
                const isLoading = getArticlesPageIsLoading(getState());
    
                if (hasMore && !isLoading) {
                    dispatch(articlesPageActions.setPage(page + 1));
                    dispatch(fetchArticlesList({}));
                }
            },
        );

 ### Services > initArticlesPage > initArticlesPage.ts:
 
    import { createAsyncThunk } from '@reduxjs/toolkit';
    import { ThunkConfig } from 'app/providers/StoreProvider';
    import { ArticleSortField, ArticleType } from 'entities/Article';
    import { SortOrder } from 'shared/types';
    import { getArticlesPageInited } from '../../selectors/articlesPageSelectors';
    import { articlesPageActions } from '../../slices/articlesPageSlice';
    import { fetchArticlesList } from '../fetchArticlesList/fetchArticlesList';
    
    export const initArticlesPage = createAsyncThunk<
        void,
        URLSearchParams,
        ThunkConfig<string>
        >(
            'articlesPage/initArticlesPage',
            async (searchParams, thunkApi) => {
                const { getState, dispatch } = thunkApi;
                const inited = getArticlesPageInited(getState());
    
                if (!inited) {
                    const orderFromUrl = searchParams.get('order') as SortOrder;
                    const sortFromUrl = searchParams.get('sort') as ArticleSortField;
                    const searchFromUrl = searchParams.get('search');
                    const typeFromUrl = searchParams.get('type') as ArticleType;
    
                    if (orderFromUrl) {
                        dispatch(articlesPageActions.setOrder(orderFromUrl));
                    }
                    if (sortFromUrl) {
                        dispatch(articlesPageActions.setSort(sortFromUrl));
                    }
                    if (searchFromUrl) {
                        dispatch(articlesPageActions.setSearch(searchFromUrl));
                    }
                    if (typeFromUrl) {
                        dispatch(articlesPageActions.setType(typeFromUrl));
                    }
    
                    dispatch(articlesPageActions.initState());
                    dispatch(fetchArticlesList({}));
                }
            },
        );

  ### Services > slices > articlesPageSlice.ts:
  
    import { createEntityAdapter, createSlice, PayloadAction } from '@reduxjs/toolkit';
    import { StateSchema } from 'app/providers/StoreProvider';
    import {
        Article, ArticleType, ArticleView, ArticleSortField,
    } from 'entities/Article';
    import { ARTICLES_VIEW_LOCALSTORAGE_KEY } from 'shared/const/localstorage';
    import { SortOrder } from 'shared/types';
    import { ArticlesPageSchema } from '../types/articlesPageSchema';
    import { fetchArticlesList } from '../../model/services/fetchArticlesList/fetchArticlesList';
    
    const articlesAdapter = createEntityAdapter<Article>({
        selectId: (article) => article.id,
    });
    
    export const getArticles = articlesAdapter.getSelectors<StateSchema>(
        (state) => state.articlesPage || articlesAdapter.getInitialState(),
    );
    
    const articlesPageSlice = createSlice({
        name: 'articlesPageSlice',
        initialState: articlesAdapter.getInitialState<ArticlesPageSchema>({
            isLoading: false,
            error: undefined,
            ids: [],
            entities: {},
            view: ArticleView.SMALL,
            page: 1,
            hasMore: true,
            _inited: false,
            limit: 9,
            sort: ArticleSortField.CREATED,
            search: '',
            order: 'asc',
            type: ArticleType.ALL,
        }),
        reducers: {
            setView: (state, action: PayloadAction<ArticleView>) => {
                state.view = action.payload;
                localStorage.setItem(ARTICLES_VIEW_LOCALSTORAGE_KEY, action.payload);
            },
            setPage: (state, action: PayloadAction<number>) => {
                state.page = action.payload;
            },
            setOrder: (state, action: PayloadAction<SortOrder>) => {
                state.order = action.payload;
            },
            setSort: (state, action: PayloadAction<ArticleSortField>) => {
                state.sort = action.payload;
            },
            setType: (state, action: PayloadAction<ArticleType>) => {
                state.type = action.payload;
            },
            setSearch: (state, action: PayloadAction<string>) => {
                state.search = action.payload;
            },
            initState: (state) => {
                const view = localStorage.getItem(ARTICLES_VIEW_LOCALSTORAGE_KEY) as ArticleView;
                state.view = view;
                state.limit = view === ArticleView.BIG ? 4 : 9;
                state._inited = true;
            },
        },
        extraReducers: (builder) => {
            builder
                .addCase(fetchArticlesList.pending, (state, action) => {
                    state.error = undefined;
                    state.isLoading = true;
    
                    if (action.meta.arg.replace) {
                        articlesAdapter.removeAll(state);
                    }
                })
                .addCase(fetchArticlesList.fulfilled, (
                    state,
                    action,
                ) => {
                    state.isLoading = false;
                    state.hasMore = action.payload.length >= state.limit;
    
                    if (action.meta.arg.replace) {
                        articlesAdapter.setAll(state, action.payload);
                    } else {
                        articlesAdapter.addMany(state, action.payload);
                    }
                })
                .addCase(fetchArticlesList.rejected, (state, action) => {
                    state.isLoading = false;
                    state.error = action.payload;
                });
        },
    });
    
    export const {
        reducer: articlesPageReducer,
        actions: articlesPageActions,
    } = articlesPageSlice;

  ### Services > types > articlesPageSchema.ts:
  
    import { EntityState } from '@reduxjs/toolkit';
    import {
        Article, ArticleView, ArticleSortField, ArticleType,
    } from 'entities/Article';
    import { SortOrder } from 'shared/types';
    
    export interface ArticlesPageSchema extends EntityState<Article> {
        isLoading?: boolean;
        error?: string;
    
        // pagination
        page: number;
        limit: number;
        hasMore: boolean;
        // filters
        view: ArticleView;
        order: SortOrder;
        sort: ArticleSortField;
        search: string;
        type: ArticleType;
    
        _inited: boolean;
    }

## Работа ЛОКАЛЬНО со своим JSON-SERVER:

    const fs = require('fs');
    const jsonServer = require('json-server');
    const path = require('path');
    
    const server = jsonServer.create();
    
    const router = jsonServer.router(path.resolve(__dirname, 'db.json'));
    
    server.use(jsonServer.defaults({}));
    server.use(jsonServer.bodyParser);
    
    // Нужно для небольшой задержки, чтобы запрос проходил не мгновенно, имитация реального апи
    server.use(async (req, res, next) => {
        await new Promise((res) => {
            setTimeout(res, 800);
        });
        next();
    });
    
    // Эндпоинт для логина
    server.post('/login', (req, res) => {
        try {
            const { username, password } = req.body;
            const db = JSON.parse(fs.readFileSync(path.resolve(__dirname, 'db.json'), 'UTF-8'));
            const { users = [] } = db;
    
            const userFromBd = users.find(
                (user) => user.username === username && user.password === password,
            );
    
            if (userFromBd) {
                return res.json(userFromBd);
            }
    
            return res.status(403).json({ message: 'User not found' });
        } catch (e) {
            console.log(e);
            return res.status(500).json({ message: e.message });
        }
    });
    
    // проверяем, авторизован ли пользователь
    // eslint-disable-next-line
    server.use((req, res, next) => {
        if (!req.headers.authorization) {
            return res.status(403).json({ message: 'AUTH ERROR' });
        }
    
        next();
    });
    
    server.use(router);
    
    // запуск сервера
    server.listen(8000, () => {
        console.log('server is running on 8000 port');
    });

## Работа с путями (Router):
### Структура директории:

    app > providers > router >    ui   >   AppRouter.tsx
                      StoreProvider        RequireAuth.tsx
                      ThemeProvider
                      ErrorBoundary

### AppRouter.tsx :

    import React, { memo, Suspense, useCallback } from 'react';
    import { Route, Routes } from 'react-router-dom';
    import { PageLoader } from 'widgets/PageLoader/PageLoader';
    import { AppRoutesProps, routeConfig } from 'shared/config/routeConfig/routeConfig';
    import { RequireAuth } from 'app/providers/router/ui/RequireAuth';
    
    const AppRouter = () => {
        const renderWithWrapper = useCallback((route: AppRoutesProps) => {
            const element = (
                <Suspense fallback={<PageLoader />}>
                    {route.element}
                </Suspense>
            );
            return (
                <Route
                    key={route.path}
                    path={route.path}
                    element={route.authOnly ? <RequireAuth>{element}</RequireAuth> : element}
                />
            );
        }, []);
    
        return (
            <Routes>
                {Object.values(routeConfig).map(renderWithWrapper)}
            </Routes>
        );
    };
    
    export default memo(AppRouter);

### RequireAuth.tsx :

    import { useSelector } from 'react-redux';
    import { getUserAuthData } from 'entities/User';
    import { Navigate, useLocation } from 'react-router-dom';
    import { RoutePath } from 'shared/config/routeConfig/routeConfig';
    
    export function RequireAuth({ children }: { children: JSX.Element }) {
        const auth = useSelector(getUserAuthData);
        const location = useLocation();
    
        if (!auth) {
            // Redirect them to the /login page, but save the current location they were
            // trying to go to when they were redirected. This allows us to send them
            // along to that page after they login, which is a nicer user experience
            // than dropping them off on the home page.
            return <Navigate to={RoutePath.main} state={{ from: location }} replace />;
        }
    
        return children;
    }

  ### КОНСТРУКЦИЯ ГЛАВНОГО ФАЙЛА С КОНФИГОМ ROUTE 
  #### Помимо этого в shared слое (shared > config > routeConfig) размещен основной конфиг. Почему в shared? Так как конфиг мы можем передать только выше! Код конфига:

      import { RouteProps } from 'react-router-dom';
      import { MainPage } from 'pages/MainPage';
      import { AboutPage } from 'pages/AboutPage';
      import { NotFoundPage } from 'pages/NotFoundPage';
      import { ProfilePage } from 'pages/ProfilePage';
      import { ArticlesPage } from 'pages/ArticlesPage';
      import { ArticleDetailsPage } from 'pages/ArticleDetailsPage';
      import { ArticleEditPage } from 'pages/ArticleEditPage';
      
      export type AppRoutesProps = RouteProps & {
          authOnly?: boolean;
      }
      
      export enum AppRoutes {
          MAIN = 'main',
          ABOUT = 'about',
          PROFILE = 'profile',
          ARTICLES = 'articles',
          ARTICLE_DETAILS = 'article_details',
          ARTICLE_CREATE = 'article_create',
          ARTICLE_EDIT = 'article_edit',
          // last
          NOT_FOUND = 'not_found',
      }
      
      export const RoutePath: Record<AppRoutes, string> = {
          [AppRoutes.MAIN]: '/',
          [AppRoutes.ABOUT]: '/about',
          [AppRoutes.PROFILE]: '/profile/', // + :id
          [AppRoutes.ARTICLES]: '/articles',
          [AppRoutes.ARTICLE_DETAILS]: '/articles/', // + :id
          [AppRoutes.ARTICLE_CREATE]: '/articles/new',
          [AppRoutes.ARTICLE_EDIT]: '/articles/:id/edit',
          // последний
          [AppRoutes.NOT_FOUND]: '*',
      };
      
      export const routeConfig: Record<AppRoutes, AppRoutesProps> = {
          [AppRoutes.MAIN]: {
              path: RoutePath.main,
              element: <MainPage />,
          },
          [AppRoutes.ABOUT]: {
              path: RoutePath.about,
              element: <AboutPage />,
          },
          [AppRoutes.PROFILE]: {
              path: `${RoutePath.profile}:id`,
              element: <ProfilePage />,
              authOnly: true,
          },
          [AppRoutes.ARTICLES]: {
              path: RoutePath.articles,
              element: <ArticlesPage />,
              authOnly: true,
          },
          [AppRoutes.ARTICLE_DETAILS]: {
              path: `${RoutePath.article_details}:id`,
              element: <ArticleDetailsPage />,
              authOnly: true,
          },
          [AppRoutes.ARTICLE_CREATE]: {
              path: `${RoutePath.article_create}`,
              element: <ArticleEditPage />,
              authOnly: true,
          },
          [AppRoutes.ARTICLE_EDIT]: {
              path: `${RoutePath.article_edit}`,
              element: <ArticleEditPage />,
              authOnly: true,
          },
          // last
          [AppRoutes.NOT_FOUND]: {
              path: RoutePath.not_found,
              element: <NotFoundPage />,
          },
      };

## ErrorBoundary:

    import React, { ErrorInfo, ReactNode, Suspense } from 'react';
    import { ErrorPage } from 'widgets/ErrorPage/ui/ErrorPage';
    
    interface ErrorBoundaryProps {
        children: ReactNode;
    }
    
    interface ErrorBoundaryState {
        hasError: boolean;
    }
    
    class ErrorBoundary
        extends React.Component<ErrorBoundaryProps, ErrorBoundaryState> {
        constructor(props: ErrorBoundaryProps) {
            super(props);
            this.state = { hasError: false };
        }
    
        static getDerivedStateFromError(error: Error) {
            // Update state so the next render will show the fallback UI.
            return { hasError: true };
        }
    
        componentDidCatch(error: Error, errorInfo: ErrorInfo) {
            // You can also log the error to an error reporting service
            console.log(error, errorInfo);
        }
    
        render() {
            const { hasError } = this.state;
            const { children } = this.props;
    
            if (hasError) {
                // You can render any custom fallback UI
                return (
                    <Suspense fallback="">
                        <ErrorPage />
                    </Suspense>
                );
            }
    
            return children;
        }
    }
    
    export default ErrorBoundary;

## Кастомный хук Троттлинг - useThrottle

    import { useCallback, useRef } from 'react';
    
    export function useThrottle(callback: (...args: any[]) => void, delay: number) {
        const throttleRef = useRef(false);
    
        return useCallback((...args: any[]) => {
            if (!throttleRef.current) {
                callback(...args);
                throttleRef.current = true;
    
                setTimeout(() => {
                    throttleRef.current = false;
                }, delay);
            }
        }, [callback, delay]);
    }

## Кастомный хук бесконечный скролл - useInfiniteScroll

    import { MutableRefObject, useEffect, useRef } from 'react';
    
    export interface UseInfiniteScrollOptions {
        callback?: () => void;
        triggerRef: MutableRefObject<HTMLElement>;
        wrapperRef: MutableRefObject<HTMLElement>;
    }
    
    export function useInfiniteScroll({ callback, wrapperRef, triggerRef }: UseInfiniteScrollOptions) {
        const observer = useRef<IntersectionObserver | null>(null);
    
        useEffect(() => {
            const wrapperElement = wrapperRef.current;
            const triggerElement = triggerRef.current;
    
            if (callback) {
                const options = {
                    root: wrapperElement,
                    rootMargin: '0px',
                    threshold: 1.0,
                };
    
                observer.current = new IntersectionObserver(([entry]) => {
                    if (entry.isIntersecting) {
                        callback();
                    }
                }, options);
    
                observer.current.observe(triggerElement);
            }
    
            return () => {
                if (observer.current && triggerElement) {
                    // eslint-disable-next-line react-hooks/exhaustive-deps
                    observer.current.unobserve(triggerElement);
                }
            };
        }, [callback, triggerRef, wrapperRef]);
    }

## Кастомный хук эффекта наведения - useHover

    import { useCallback, useMemo, useState } from 'react';
    
    interface UseHoverBind {
        onMouseEnter: () => void;
        onMouseLeave: () => void;
    }
    
    type UseHoverResult = [boolean, UseHoverBind]
    
    export const useHover = () => {
        const [isHover, setIsHover] = useState(false);
    
        const onMouseEnter = useCallback(() => {
            setIsHover(true);
        }, []);
    
        const onMouseLeave = useCallback(() => {
            setIsHover(false);
        }, []);
    
        return useMemo(() => [
            isHover,
            {
                onMouseEnter,
                onMouseLeave,
            },
        ], [isHover, onMouseEnter, onMouseLeave]);
    };

  ## Кастомный хук debounce - useDebounce
  
    import { MutableRefObject, useCallback, useRef } from 'react';
    
    export function useDebounce(callback: (...args: any[]) => void, delay: number) {
        const timer = useRef() as MutableRefObject<any>;
    
        return useCallback((...args: any[]) => {
            if (timer.current) {
                clearTimeout(timer.current);
            }
            timer.current = setTimeout(() => {
                callback(...args);
            }, delay);
        }, [callback, delay]);
    }

## Функция для наложения стилей на модуль - classNames

    export type Mods = Record<string, boolean | string | undefined>
    
    export function classNames(
        cls: string,
        mods: Mods = {},
        additional: Array<string | undefined> = [],
    ): string {
        return [
            cls,
            ...additional.filter(Boolean),
            ...Object.entries(mods)
                .filter(([_, value]) => Boolean(value))
                .map(([className]) => className),
        ]
            .join(' ');
    }

 ### Как ей пользоваться?
 
    export const ArticleCodeBlockComponent = memo((props: ArticleCodeBlockComponentProps) => {
        const { className, block } = props;
        const { t } = useTranslation();
    
        return (
            <div className={classNames(cls.ArticleCodeBlockComponent, {}, [className])}>
                <Code text={block.code} />
            </div>
        );
    });

## Пример написания README.md

    ## Запуск проекта
    
    ```
    npm install - устанавливаем зависимости
    npm run start:dev или npm run start:dev:vite - запуск сервера + frontend проекта в dev режиме
    ```
    
    ----
    
    ## Скрипты
    
    - `npm run start` - Запуск frontend проекта на webpack dev server
    - `npm run start:vite` - Запуск frontend проекта на vite
    - `npm run start:dev` - Запуск frontend проекта на webpack dev server + backend
    - `npm run start:dev:vite` - Запуск frontend проекта на vite + backend
    - `npm run start:dev:server` - Запуск backend сервера
    - `npm run build:prod` - Сборка в prod режиме
    - `npm run build:dev` - Сборка в dev режиме (не минимизирован)
    - `npm run lint:ts` - Проверка ts файлов линтером
    - `npm run lint:ts:fix` - Исправление ts файлов линтером
    - `npm run lint:scss` - Проверка scss файлов style линтером
    - `npm run lint:scss:fix` - Исправление scss файлов style линтером
    - `npm run test:unit` - Хапуск unit тестов с jest
    - `npm run test:ui` - Хапуск скриншотных тестов с loki
    - `npm run test:ui:ok` - Подтверждение новых скриншотов
    - `npm run test:ui:ci` - Запуск скриншотных тестов в CI
    - `npm run test:ui:report` - Генерация полного отчета для скриншотных тестов
    - `npm run test:ui:json` - Генерация json отчета для скриншотных тестов
    - `npm run test:ui:html` - Генерация HTML отчета для скриншотных тестов
    - `npm run storybook` - запуск Storybook
    - `npm run storybook:build` - Сборка storybook билда
    - `npm run prepare` - прекоммит хуки
    - `npm run generate:slice` - Скрипт для генерации FSD слайсов
    
    ----
    
    ## Архитектура проекта
    
    Проект написан в соответствии с методологией Feature sliced design
    
    Ссылка на документацию - [feature sliced design](https://feature-sliced.design/docs/get-started/tutorial)
    
    ----
    
    ## Работа с переводами
    
    В проекте используется библиотека i18next для работы с переводами.
    Файлы с переводами хранятся в public/locales.
    
    Для комфортной работы рекомендуем установить плагин для webstorm/vscode
    
    Документация i18next - [https://react.i18next.com/](https://react.i18next.com/)
    
    ----
    
    ## Тесты
    
    В проекте используются 4 вида тестов:
    1) Обычные unit тесты на jest - `npm run test:unit`
    2) Тесты на компоненты с React testing library -`npm run test:unit`
    3) Скриншотное тестирование с loki `npm run test:ui`
    4) e2e тестирование с Cypress `npm run test:e2e`
    
    Подробнее о тестах - [документация тестирование](/docs/tests.md)
    
    ----
    
    ## Линтинг
    
    В проекте используется eslint для проверки typescript кода и stylelint для проверки файлов со стилями.
    
    Также для строгого контроля главных архитектурных принципов
    используется собственный eslint plugin *eslint-plugin-ulbi-tv-plugin*,
    который содержит 3 правила
    1) path-checker - запрещает использовать абсолютные импорты в рамках одного модуля
    2) layer-imports - проверяет корректность использования слоев с точки зрения FSD
       (например widgets нельзя использовать в features и entitites)
    3) public-api-imports - разрешает импорт из других модулей только из public api. Имеет auto fix
    
    ##### Запуск линтеров
    - `npm run lint:ts` - Проверка ts файлов линтером
    - `npm run lint:ts:fix` - Исправление ts файлов линтером
    - `npm run lint:scss` - Проверка scss файлов style линтером
    - `npm run lint:scss:fix` - Исправление scss файлов style линтером
    
    ----
    ## Storybook
    
    В проекте для каждого компонента описываются стори-кейсы.
    Запросы на сервер мокаются с помощью storybook-addon-mock.
    
    Файл со сторикейсами создает рядом с компонентом с расширением .stories.tsx
    
    Запустить сторибук можно командой:
    - `npm run storybook`
    
    Подробнее о [Storybook](/docs/storybook.md)
    
    Пример:
    
    ```typescript jsx
    import React from 'react';
    import { ComponentStory, ComponentMeta } from '@storybook/react';
    
    import { ThemeDecorator } from '@/shared/config/storybook/ThemeDecorator/ThemeDecorator';
    import { Button, ButtonSize, ButtonTheme } from './Button';
    import { Theme } from '@/shared/const/theme';
    
    export default {
        title: 'shared/Button',
        component: Button,
        argTypes: {
            backgroundColor: { control: 'color' },
        },
    } as ComponentMeta<typeof Button>;
    
    const Template: ComponentStory<typeof Button> = (args) => <Button {...args} />;
    
    export const Primary = Template.bind({});
    Primary.args = {
        children: 'Text',
    };
    
    export const Clear = Template.bind({});
    Clear.args = {
        children: 'Text',
        theme: ButtonTheme.CLEAR,
    };
    ```
    
    
    ----
    
    ## Конфигурация проекта
    
    Для разработки проект содержит 2 конфига:
    1. Webpack - ./config/build
    2. vite - vite.config.ts
    
    Оба сборщика адаптированы под основные фичи приложения.
    
    Вся конфигурация хранится в /config
    - /config/babel - babel
    - /config/build - конфигурация webpack
    - /config/jest - конфигурация тестовой среды
    - /config/storybook - конфигурация сторибука
    
    В папке `scripts` находятся различные скрипты для рефакторинга\упрощения написания кода\генерации отчетов и тд.
    
    ----
    
    ## CI pipeline и pre commit хуки
    
    Конфигурация github actions находится в /.github/workflows.
    В ci прогоняются все виды тестов, сборка проекта и сторибука, линтинг.
    
    В прекоммит хуках проверяем проект линтерами, конфиг в /.husky
    
    ----
    
    ### Работа с данными
    
    Взаимодействие с данными осуществляется с помощью redux toolkit.
    По возможности переиспользуемые сущности необходимо нормализовать с помощью EntityAdapter
    
    Запросы на сервер отправляются с помощью [RTK query](/src/shared/api/rtkApi.ts)
    
    Для асинхронного подключения редюсеров (чтобы не тянуть их в общий бандл) используется
    [DynamicModuleLoader](/src/shared/lib/components/DynamicModuleLoader/DynamicModuleLoader.tsx)
    
    ----
    
    
    ## Сущности (entities)
    
    - [Article](/src/entities/Article)
    - [Comment](/src/entities/Comment)
    - [Counter](/src/entities/Counter)
    - [Country](/src/entities/Country)
    - [Currency](/src/entities/Currency)
    - [Notification](/src/entities/Notification)
    - [Profile](/src/entities/Profile)
    - [Rating](/src/entities/Rating)
    - [User](/src/entities/User)
    
    ## Фичи (features)
    
    - [addCommentForm](/src/features/addCommentForm)
    - [articleEditForm](/src/features/articleEditForm)
    - [articleRating](/src/features/articleRating)
    - [articleRecommendationsList](/src/features/articleRecommendationsList)
    - [AuthByUsername](/src/features/AuthByUsername)
    - [avatarDropdown](/src/features/avatarDropdown)
    - [editableProfileCard](/src/features/editableProfileCard)
    - [LangSwitcher](/src/features/LangSwitcher)
    - [notificationButton](/src/features/notificationButton)
    - [profileRating](/src/features/profileRating)
    - [ThemeSwitcher](/src/features/ThemeSwitcher)
    - [UI](/src/features/UI)
    


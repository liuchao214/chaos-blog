---
title: webpack配置小记
date: 2020-04-17 18:20:47
index_img: /images/webpack.jpeg
banner_img: /images/20191231163323.jpg
tags: [webpack]
categories: [工程化]
---

## 多入口配置
<!-- more -->

```js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { srcPath, distPath } = require('./paths')

module.exports = {
    entry: {
        index: path.join(srcPath, 'index.js'),
        other: path.join(srcPath, 'other.js')
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                loader: ['babel-loader'],
                include: srcPath,
                exclude: /node_modules/
            },
            // {
            //     test: /\.css$/,
            //     // loader 的执行顺序是：从后往前
            //     loader: ['style-loader', 'css-loader']
            // },
            {
                test: /\.css$/,
                // loader 的执行顺序是：从后往前
                loader: ['style-loader', 'css-loader', 'postcss-loader'] // 加了 postcss
            },
            {
                test: /\.less$/,
                // 增加 'less-loader' ，注意顺序
                loader: ['style-loader', 'css-loader', 'less-loader']
            }
        ]
    },
    plugins: [
        // new HtmlWebpackPlugin({
        //     template: path.join(srcPath, 'index.html'),
        //     filename: 'index.html'
        // })

        // 多入口 - 生成 index.html
        new HtmlWebpackPlugin({
            template: path.join(srcPath, 'index.html'),
            filename: 'index.html',
            // chunks 表示该页面要引用哪些 chunk （即上面的 index 和 other），默认全部引用
            chunks: ['index']  // 只引用 index.js
        }),
        // 多入口 - 生成 other.html
        new HtmlWebpackPlugin({
            template: path.join(srcPath, 'other.html'),
            filename: 'other.html',
            chunks: ['other']  // 只引用 other.js
        })
    ]
}

```
## CSS压缩

```js
const path = require('path')
const webpack = require('webpack')
const { smart } = require('webpack-merge')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const TerserJSPlugin = require('terser-webpack-plugin')
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin')
const webpackCommonConf = require('./webpack.common.js')
const { srcPath, distPath } = require('./paths')  

module.exports = smart(webpackCommonConf, {
    mode: 'production',
    output: {
        // filename: 'bundle.[contentHash:8].js',  // 打包代码时，加上 hash 戳
        filename: '[name].[contentHash:8].js', // name 即多入口时 entry 的 key
        path: distPath,
        // publicPath: 'http://cdn.abc.com'  // 修改所有静态文件 url 的前缀（如 cdn 域名），这里暂时用不到
    },
    module: {
        rules: [
            // 图片 - 考虑 base64 编码的情况
            {
                test: /\.(png|jpg|jpeg|gif)$/,
                use: {
                    loader: 'url-loader',
                    options: {
                        // 小于 5kb 的图片用 base64 格式产出
                        // 否则，依然延用 file-loader 的形式，产出 url 格式
                        limit: 5 * 1024,

                        // 打包到 img 目录下
                        outputPath: '/img1/',

                        // 设置图片的 cdn 地址（也可以统一在外面的 output 中设置，那将作用于所有静态资源）
                        // publicPath: 'http://cdn.abc.com'
                    }
                }
            },
            // 抽离 css
            {
                test: /\.css$/,
                loader: [
                    MiniCssExtractPlugin.loader,  // 注意，这里不再用 style-loader
                    'css-loader',
                    'postcss-loader'
                ]
            },
            // 抽离 less --> css
            {
                test: /\.less$/,
                loader: [
                    MiniCssExtractPlugin.loader,  // 注意，这里不再用 style-loader
                    'css-loader',
                    'less-loader',
                    'postcss-loader'
                ]
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(), // 会默认清空 output.path 文件夹
        new webpack.DefinePlugin({
            // window.ENV = 'production'
            ENV: JSON.stringify('production')
        }),

        // 抽离 css 文件
        new MiniCssExtractPlugin({
            filename: 'css/main.[contentHash:8].css'
        })
    ],

    optimization: {
        // 压缩 css
        minimizer: [new TerserJSPlugin({}), new OptimizeCSSAssetsPlugin({})],
    }
})

```
## 抽离公共代码

```js
const path = require('path')
const webpack = require('webpack')
const { smart } = require('webpack-merge')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const TerserJSPlugin = require('terser-webpack-plugin')
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin')
const webpackCommonConf = require('./webpack.common.js')
const { srcPath, distPath } = require('./paths')

module.exports = smart(webpackCommonConf, {
    mode: 'production',
    output: {
        // filename: 'bundle.[contentHash:8].js',  // 打包代码时，加上 hash 戳
        filename: '[name].[contentHash:8].js', // name 即多入口时 entry 的 key
        path: distPath,
        // publicPath: 'http://cdn.abc.com'  // 修改所有静态文件 url 的前缀（如 cdn 域名），这里暂时用不到
    },
    module: {
        rules: [
            // 图片 - 考虑 base64 编码的情况
            {
                test: /\.(png|jpg|jpeg|gif)$/,
                use: {
                    loader: 'url-loader',
                    options: {
                        // 小于 5kb 的图片用 base64 格式产出
                        // 否则，依然延用 file-loader 的形式，产出 url 格式
                        limit: 5 * 1024,

                        // 打包到 img 目录下
                        outputPath: '/img1/',

                        // 设置图片的 cdn 地址（也可以统一在外面的 output 中设置，那将作用于所有静态资源）
                        // publicPath: 'http://cdn.abc.com'
                    }
                }
            },
            // 抽离 css
            {
                test: /\.css$/,
                loader: [
                    MiniCssExtractPlugin.loader,  // 注意，这里不再用 style-loader
                    'css-loader',
                    'postcss-loader'
                ]
            },
            // 抽离 less
            {
                test: /\.less$/,
                loader: [
                    MiniCssExtractPlugin.loader,  // 注意，这里不再用 style-loader
                    'css-loader',
                    'less-loader',
                    'postcss-loader'
                ]
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(), // 会默认清空 output.path 文件夹
        new webpack.DefinePlugin({
            // window.ENV = 'production'
            ENV: JSON.stringify('production')
        }),

        // 抽离 css 文件
        new MiniCssExtractPlugin({
            filename: 'css/main.[contentHash:8].css'
        })
    ],

    optimization: {
        // 压缩 css
        minimizer: [new TerserJSPlugin({}), new OptimizeCSSAssetsPlugin({})],

        // 分割代码块
        splitChunks: {
            chunks: 'all',
            /**
             * initial 入口 chunk，对于异步导入的文件不处理
                async 异步 chunk，只对异步导入的文件处理
                all 全部 chunk
             */

            // 缓存分组
            cacheGroups: {
                // 第三方模块
                vendor: {
                    name: 'vendor', // chunk 名称
                    priority: 1, // 权限更高，优先抽离，重要！！！
                    test: /node_modules/,
                    minSize: 0,  // 大小限制
                    minChunks: 1  // 最少复用过几次
                },

                // 公共的模块
                common: {
                    name: 'common', // chunk 名称
                    priority: 0, // 优先级
                    minSize: 0,  // 公共模块的大小限制
                    minChunks: 2  // 公共模块最少复用过几次
                }
            }
        }
    }
})

```

## vue.config.js

```js
/**
 * @file vue-cli4基础配置
 * 配置参考: https://cli.vuejs.org/zh/config/
 */

'use strict';
const fs = require('fs');
const path = require('path');
const webpack = require('webpack');
const FilterWarningsPlugin = require('webpack-filter-warnings-plugin');
const StyleLintPlugin = require('stylelint-webpack-plugin');

const assetsPath = assetsPath => {
    return path.posix.join('static', assetsPath);
};
const resolve = dir => {
    return path.join(__dirname, dir);
};

module.exports = {
    publicPath: '/',
    chainWebpack: config => {
        // px自动转换为rem
        config.module
        .rule('less')
        .test(/\.less$/)
        .oneOf('vue')
        .use('px2rem-loader')
        .loader('px2rem-loader')
        .before('postcss-loader')
        .options({remUnit: 16, remPrecision: 8})
        .end();
        // svg-loader
        const svgRule = config.module.rule('svg');
        svgRule.uses.clear();
        svgRule
        .test(/\.(svg)(\?.*)?$/)
        .use('url-loader')
        .loader('url-loader')
        .options({
            limit: 4096,
            name: assetsPath('img/[name].[hash:7].[ext]')
        })
        .end();
        // moment引入（繁简英）语言包；
        config
        .plugin('context-replacement')
        .use(webpack.ContextReplacementPlugin, [
            /moment[\\\/]locale$/,
            /^\.\/(zh-cn|en|zh-tw)$/
        ]);
        // 去除CSS顺序造成的提示信息
        config.plugin('filter-warnings').use(FilterWarningsPlugin, [
            {
                exclude: /mini-css-extract-plugin[^]*Conflicting order between:/
            }
        ]);
        // 样式表校验
        config.plugin('style-lint').use(StyleLintPlugin, [
            {
                files: ['src/**/*.{vue,html,css,less}'],
                fix: true
            }
        ]);
        // 全局变量配置
        /*config.plugin('provide-plugin').use(webpack.ProvidePlugin, [
            {
                Hls: 'hls'
            }
        ]);*/
        // 设置页面基础配置
        config.plugin('html').tap(args => {
            args[0].title = 'BAIDU-CVS';
            args[0].meta = {
                keywords: 'BAIDU-CVS',
                description: 'BAIDU-CVS'
            };
            return args;
        });
    },
    outputDir: './dist',
    assetsDir: 'static',
    css: {
        loaderOptions: {
            less: {
                lessOptions: {
                    modifyVars: {
                        'primary-color': '#3B48FA',
                        'border-radius-base': '4px',
                        'layout-body-background': '#FAFAFC',
                        'layout-sider-background': '#0C2146',
                        'label-color': 'fade(#0C2146, 85%)',
                        'font-size-base': '0.875rem',
                        'font-size-lg': '1rem',
                        'font-size-sm': '0.75rem',
                        'text-color': 'fade(#0C2146, 65%)',
                        'text-color-dark': 'fade(#0C2146, 85%)',
                        'input-placeholder-color': 'fade(#0C2146, 25%)',
                        'border-color-base': '#EDEEF1',
                        'menu-dark-bg': '#0C2146',
                        'menu-dark-item-active-bg': '#0C2146',
                        'menu-dark-color': 'fade(#fff, 60%)',
                        'modal-mask-bg': 'fade(#0C2146, 50%)',
                        'menu-dark-submenu-bg': '#091a37',
                        'modal-heading-color': 'fade(#0C2146, 85%)'
                    },
                    javascriptEnabled: true
                }
            }
        }
    },
    configureWebpack: {
        resolve: {
            alias: {
                '@': resolve('src'),
                vue$: 'vue/dist/vue.esm.js'
                // hls: path.resolve(__dirname, 'public/flash/hls.min.js')
            }
        },
        module: {
            noParse: /^(vue|vue-router|vuex|vuex-router-sync|lodash|moment)$/
        },
        // 警告 webpack 的性能提示
        performance: {
            hints: 'warning',
            // 入口起点的最大体积
            maxEntrypointSize: Infinity,
            // 生成文件的最大体积
            maxAssetSize:
                process.env.NODE_ENV === 'production'
                    ? 2 * 1024 * 1024 // 2.0M
                    : Infinity,
            // 只给出 js 文件的性能提示
            assetFilter: assetFilename => {
                return assetFilename.endsWith('.js');
            }
        },
        optimization: {
            runtimeChunk: {
                name: 'manifest'
            },
            splitChunks: {
                chunks: 'initial',
                cacheGroups: {
                    'ant-design-vue': {
                        name: 'ant-design-vue',
                        test: /ant-design-vue[\\\/]/,
                        priority: 10,
                        chunks: 'initial'
                    },
                    'antv-g6': {
                        name: 'antv-g6',
                        test: /@antv[\\\/]g6[\\\/]/,
                        priority: 8,
                        chunks: 'all'
                    },
                    echarts: {
                        name: 'echarts',
                        test: /echarts[\\\/]/,
                        priority: 6,
                        chunks: 'all'
                    },
                    swiper: {
                        name: 'swiper',
                        test: /swiper[\\\/]/,
                        priority: 4,
                        chunks: 'initial'
                    },
                    vendors: {
                        name: 'vendors',
                        test: /[\\\/]node_modules[\\\/]/,
                        priority: 0,
                        chunks: 'initial'
                    }
                }
            }
        }
    },
    productionSourceMap: false,
    devServer: {
        proxy:{
            // ...
        },
        open: true,
        hot: true,
        port: '****',
        overlay: {
            errors: true,
            warnings: true
        }
    }
};
```


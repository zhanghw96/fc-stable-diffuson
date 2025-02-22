<template>
    <div>
        <template v-for="(item) in prompts" :key="item.name">
            <physton-prompt v-if="item.$textarea" :id="item.id" :ref="item.id" :name="item.name"
                            :neg="item.neg" :textarea="item.$textarea" :steps="item.$steps"
                            v-model:language-code="languageCode"
                            :translate-apis="translateApis" :languages="languages"
                            :history-key="item.historyKey"
                            @click:show-history="onShowHistory(item.id, $event)"
                            :favorite-key="item.favoriteKey"
                            @refresh-favorites="onRefreshFavorites"
                            @click:show-favorite="onShowFavorite(item.id, $event)"
                            v-model:can-one-translate="canOneTranslate"
                            v-model:auto-translate="autoTranslate"
                            v-model:auto-translate-to-english="autoTranslateToEnglish"
                            v-model:auto-translate-to-local="autoTranslateToLocal"
                            v-model:auto-remove-space="autoRemoveSpace"
                            v-model:auto-remove-last-comma="autoRemoveLastComma"
                            v-model:auto-keep-weight-zero="autoKeepWeightZero"
                            v-model:auto-keep-weight-one="autoKeepWeightOne"
                            v-model:auto-break-before-wrap="autoBreakBeforeWrap"
                            v-model:auto-break-after-wrap="autoBreakAfterWrap"
                            :hide-default-input="item.hideDefaultInput"
                            @update:hide-default-input="onUpdateHideDefaultInput(item.id, $event)"
                            :hide-panel="item.hidePanel"
                            @update:hide-panel="onUpdateHidePanel(item.id, $event)"
                            v-model:enable-tooltip="enableTooltip"
                            v-model:translate-api="translateApi"
                            :translate-api-config="translateApiConfig"
                            @click:translate-api="onTranslateApiClick"
                            @click:prompt-format="onPromptFormatClick"
                            v-model:tag-complete-file="tagCompleteFile"
                            v-model:only-csv-on-auto="onlyCsvOnAuto"
                            @click:select-language="onSelectLanguageClick"
                            @click:select-theme="onSelectThemeClick"
                            @click:show-chatgpt="onShowChatgpt(item.id, $event)"
                            :extra-networks="extraNetworks"
                            :loras="loras"
                            :lycos="lycos"
                            :embeddings="embeddings"
                            :version="version"
                            :latest-version="latestVersion"
                            :is-latest-version="isLatestVersion"
                            @click:show-about="onShowAbout"
                            :theme="theme"
                            @click:switch-theme="onSwitchTheme"
            ></physton-prompt>
        </template>
        <translate-setting ref="translateSetting" v-model:language-code="languageCode"
                           :translate-apis="translateApis" :languages="languages"
                           @forceUpdate:translateApi="updateTranslateApiConfig"
                           v-model:tag-complete-file="tagCompleteFile"
                           v-model:only-csv-on-auto="onlyCsvOnAuto"
                           v-model:translate-api="translateApi"></translate-setting>
        <select-language ref="selectLanguage" v-model:language-code="languageCode"
                         :translate-apis="translateApis"
                         :languages="languages"
                         v-model:translate-api="translateApi"
                         v-model:tag-complete-file="tagCompleteFile"
                         v-model:only-csv-on-auto="onlyCsvOnAuto"></select-language>
        <prompt-format ref="promptFormat" v-model:language-code="languageCode"
                       :translate-apis="translateApis"
                       :languages="languages"
                       v-model:auto-remove-space="autoRemoveSpace"
                       v-model:auto-remove-last-comma="autoRemoveLastComma"
                       v-model:auto-keep-weight-zero="autoKeepWeightZero"
                       v-model:auto-keep-weight-one="autoKeepWeightOne"
                       v-model:auto-break-before-wrap="autoBreakBeforeWrap"
                       v-model:auto-break-after-wrap="autoBreakAfterWrap"></prompt-format>
        <history ref="history" v-model:language-code="languageCode"
                 :translate-apis="translateApis" :languages="languages"
                 v-model:tag-complete-file="tagCompleteFile"
                 v-model:only-csv-on-auto="onlyCsvOnAuto"
                 @refresh-favorites="onRefreshFavorites"
                 @use="onUseHistory"/>
        <favorite ref="favorite" v-model:language-code="languageCode"
                  :translate-apis="translateApis" :languages="languages"
                  v-model:tag-complete-file="tagCompleteFile"
                  v-model:only-csv-on-auto="onlyCsvOnAuto"
                  @use="onUseFavorite"></favorite>
        <extension-css ref="extensionCss" v-model:language-code="languageCode"
                       :translate-apis="translateApis" :languages="languages"/>
        <packages-state ref="packagesState" v-model:language-code="languageCode"
                        :translate-apis="translateApis" :languages="languages"
                        @click:select-language="onSelectLanguageClick"
                        :packages-state="packagesState" :python="python"/>
        <chatgpt-prompt ref="chatgptPrompt" v-model:language-code="languageCode"
                        :translate-apis="translateApis" :languages="languages"
                        @use="onUseChatgpt" />
        <about ref="about" v-model:language-code="languageCode"
               :translate-apis="translateApis" :languages="languages" />

        <div class="physton-paste-popup" v-if="showPastePopup" @click="closePastePopup">
            <div class="paste-popup-main" @click.stop>
                <div class="paste-popup-close" @click="closePastePopup">
                    <icon-svg name="close"/>
                </div>
                <div class="paste-popup-title">{{ pasteTitle }}</div>
                <div class="paste-popup-body">
                    <textarea class="paste-content" v-model="pasteContent"
                              :placeholder="getLang('please_enter_the_content_here')"></textarea>
                    <div v-if="!pasteLoading" class="paste-submit" @click="onClickPasteSubmit">Submit</div>
                    <div v-else class="paste-submit">
                        <icon-svg name="loading"/>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import PhystonPrompt from "./components/phystonPrompt.vue"
import TranslateSetting from "@/components/translateSetting.vue";
import common from "@/utils/common";
import SelectLanguage from "@/components/selectLanguage.vue";
import Favorite from "@/components/favorite.vue";
import History from "@/components/history.vue";
import IconSvg from "@/components/iconSvg.vue";
import ExtensionCss from "@/components/extensionCss.vue";
import PromptFormat from "@/components/promptFormat.vue";
import PackagesState from "@/components/packagesState.vue";
import ChatgptPrompt from "@/components/chatgptPrompt.vue";
import About from "@/components/about.vue";
import globals from "../globals";

export default {
    name: 'App',
    components: {
        About,
        ChatgptPrompt,
        PackagesState,
        PromptFormat,
        ExtensionCss,
        IconSvg,
        History,
        Favorite,
        SelectLanguage,
        TranslateSetting,
        PhystonPrompt,
    },
    mixins: [],
    data() {
        return {
            prompts: [
                {
                    tab: 'tab_txt2img',
                    prompt: 'txt2img_prompt',
                    counter: 'txt2img_token_counter',
                    button: 'txt2img_token_button',
                    steps: 'txt2img_steps',
                    historyKey: 'txt2img',
                    favoriteKey: 'txt2img',
                    $prompt: null,
                    $textarea: null,
                    $steps: null,
                    name: 'txt2img_prompt',
                    neg: false,
                    hideDefaultInputKey: 'txt2ImgHideDefaultInput',
                    hideDefaultInput: false,
                    hidePanelKey: 'txt2ImgHidePanel',
                    hidePanel: false,
                    id: 'phystonPrompt_txt2img_prompt'
                },
                {
                    tab: 'tab_txt2img',
                    prompt: 'txt2img_neg_prompt',
                    counter: 'txt2img_negative_token_counter',
                    button: 'txt2img_negative_token_button',
                    steps: 'txt2img_steps',
                    historyKey: 'txt2img_neg',
                    favoriteKey: 'txt2img_neg',
                    $prompt: null,
                    $textarea: null,
                    $steps: null,
                    name: 'txt2img_neg_prompt',
                    neg: true,
                    hideDefaultInputKey: 'txt2ImgNegHideDefaultInput',
                    hideDefaultInput: false,
                    hidePanelKey: 'txt2ImgNegHidePanel',
                    hidePanel: false,
                    id: 'phystonPrompt_txt2img_neg_prompt'
                },
                {
                    tab: 'tab_img2img',
                    prompt: 'img2img_prompt',
                    counter: 'img2img_token_counter',
                    button: 'img2img_token_button',
                    steps: 'img2img_steps',
                    historyKey: 'img2img',
                    favoriteKey: 'img2img',
                    $prompt: null,
                    $textarea: null,
                    $steps: null,
                    name: 'img2img_prompt',
                    neg: false,
                    hideDefaultInputKey: 'img2ImgHideDefaultInput',
                    hideDefaultInput: false,
                    hidePanelKey: 'img2ImgHidePanel',
                    hidePanel: false,
                    id: 'phystonPrompt_img2img_prompt'
                },
                {
                    tab: 'tab_img2img',
                    prompt: 'img2img_neg_prompt',
                    counter: 'img2img_negative_token_counter',
                    button: 'img2img_negative_token_button',
                    steps: 'img2img_steps',
                    historyKey: 'img2img_neg',
                    favoriteKey: 'img2img_neg',
                    $prompt: null,
                    $textarea: null,
                    $steps: null,
                    name: 'img2img_neg_prompt',
                    neg: true,
                    hideDefaultInputKey: 'img2ImgNegHideDefaultInput',
                    hideDefaultInput: false,
                    hidePanelKey: 'img2ImgNegHidePanel',
                    hidePanel: false,
                    id: 'phystonPrompt_img2img_neg_prompt'
                },
            ],
            languageCode: '',
            languages: {},
            translateApis: [],
            translateApi: '',
            translateApiConfig: {},
            canOneTranslate: false,
            autoTranslate: false,
            autoTranslateToEnglish: false,
            autoTranslateToLocal: false,
            autoRemoveSpace: true,
            autoRemoveLastComma: false,
            autoKeepWeightZero: false,
            autoKeepWeightOne: false,
            autoBreakBeforeWrap: false,
            autoBreakAfterWrap: false,
            // hideDefaultInput: false,
            enableTooltip: true,
            tagCompleteFile: '',
            onlyCsvOnAuto: false,

            startWatchSave: false,

            pasteBtn: null,
            showPastePopup: false,
            pasteTitle: '',
            pasteContent: '',
            pasteLoading: false,

            historyCurrentPrompt: '',
            favoriteCurrentPrompt: '',
            chatgptCurrentPrompt: '',

            extraNetworks: [],
            loras: [],
            lycos: [],
            embeddings: [],

            python: '',
            packagesState: [],

            version: '',
            latestVersion: '',
            isLatestVersion: true,

            theme: 'dark',
        }
    },
    watch: {
        languageCode: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                console.log('onLanguageCodeChange', val)
                this.canOneTranslate = common.canOneTranslate(this.languageCode)
                this.gradioAPI.setData('languageCode', val).then(data => {
                }).catch(err => {
                })
            },
            immediate: false,
        },
        autoTranslateToEnglish: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                console.log('onAutoTranslateToEnglishChange', val)
                this.gradioAPI.setData('autoTranslateToEnglish', val).then(data => {
                }).catch(err => {
                })
            },
            immediate: false,
        },
        autoTranslateToLocal: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                console.log('onAutoTranslateToLocalChange', val)
                this.gradioAPI.setData('autoTranslateToLocal', val).then(data => {
                }).catch(err => {
                })
            },
            immediate: false,
        },
        autoTranslate: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                this.autoTranslateToEnglish = this.autoTranslate
                this.autoTranslateToLocal = this.autoTranslate
                console.log('onAutoTranslateChange', val)
                this.gradioAPI.setData('autoTranslate', val).then(data => {
                }).catch(err => {
                })
            },
            immediate: false,
        },
        autoRemoveSpace: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                console.log('onAutoRemoveSpaceChange', val)
                this.gradioAPI.setData('autoRemoveSpace', val).then(data => {
                    this.prompts.forEach(item => {
                        this.$refs[item.id][0].updatePrompt()
                    })
                }).catch(err => {
                })
            },
            immediate: false,
        },
        autoRemoveLastComma: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                console.log('onAutoRemoveLastCommaChange', val)
                this.gradioAPI.setData('autoRemoveLastComma', val).then(data => {
                    this.prompts.forEach(item => {
                        this.$refs[item.id][0].updatePrompt()
                    })
                }).catch(err => {
                })
            },
            immediate: false,
        },
        autoKeepWeightZero: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                console.log('onAutoKeepWeightZeroChange', val)
                this.gradioAPI.setData('autoKeepWeightZero', val).then(data => {
                }).catch(err => {
                })
            },
            immediate: false,
        },
        autoKeepWeightOne: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                console.log('onAutoKeepWeightOneChange', val)
                this.gradioAPI.setData('autoKeepWeightOne', val).then(data => {
                }).catch(err => {
                })
            },
            immediate: false,
        },
        autoBreakBeforeWrap: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                console.log('onAutoBreakBeforeWrap', val)
                this.gradioAPI.setData('autoBreakBeforeWrap', val).then(data => {
                    this.prompts.forEach(item => {
                        this.$refs[item.id][0].updatePrompt()
                    })
                }).catch(err => {
                })
            },
            immediate: false,
        },
        autoBreakAfterWrap: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                console.log('onAutoBreakAfterWrap', val)
                this.gradioAPI.setData('autoBreakAfterWrap', val).then(data => {
                    this.prompts.forEach(item => {
                        this.$refs[item.id][0].updatePrompt()
                    })
                }).catch(err => {
                })
            },
            immediate: false,
        },
        /*hideDefaultInput: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                console.log('onHideDefaultInputChange', val)
                this.prompts.forEach(item => {
                    item.$prompt.parentElement.parentElement.style.display = val ? 'none' : 'flex'
                })
                this.gradioAPI.setData('hideDefaultInput', val).then(data => {
                }).catch(err => {
                })
            },
            immediate: false,
        },*/
        enableTooltip: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                localStorage.setItem('phystonPromptEnableTooltip', val ? 'true' : 'false')
                this.updateTippyState()
                this.gradioAPI.setData('enableTooltip', val).then(data => {
                }).catch(err => {
                })
            },
            immediate: false,
        },
        translateApi: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                console.log('onTranslateApiChange', val, oldVal)
                this.updateTranslateApiConfig()
                this.gradioAPI.setData('translateApi', val).then(data => {
                }).catch(err => {
                })
            },
            immediate: false,
        },
        tagCompleteFile: {
            handler: function (val, oldVal) {
                if (!this.startWatchSave) return
                console.log('onTagCompleteFileChange', val, oldVal)
                this.gradioAPI.setData('tagCompleteFile', val).then(data => {
                }).catch(err => {
                })
            },
            immediate: false,
        },
        onlyCsvOnAuto() {
            if (!this.startWatchSave) return
            console.log('onOnlyCsvOnAutoChange', this.onlyCsvOnAuto)
            this.gradioAPI.setData('onlyCsvOnAuto', this.onlyCsvOnAuto).then(data => {
            }).catch(err => {
            })
        },
    },
    mounted() {
        common.loadCSS('toastr.min.css', 'physton-prompt-toastr', true, true, false)
        common.loadCSS('tippy.css', 'physton-prompt-tippy', true, true, false)
        common.loadCSS('main.min.css', 'physton-prompt-main', true)

        const urlParams = new URLSearchParams(window.location.search);
        let theme = urlParams.get("__theme")
        if (['dark', 'light'].includes(theme)) {
            this.theme = theme
            if (!common.gradioApp().classList.contains(this.theme)) {
                common.gradioApp().classList.add(this.theme)
            }
        }

        // sd-webui-lobe-theme
        setTimeout(() => {
            if (document.querySelector('meta[content="LobeHub"]')) {
                common.gradioApp().classList.add("physton-prompt-lobehub")
            }
        }, 3000)


        this.gradioAPI.getConfig().then(res => {
            console.log('config:', res)
            this.languageCode = res.i18n.default
            this.translateApi = res.translate_apis.default
            this.translateApis = res.translate_apis.apis
            this.python = res.python
            this.packagesState = res.packages_state
            let languages = {}
            res.i18n.languages.forEach(lang => {
                languages[lang.code] = lang
            })
            this.languages = languages
            this.init()
        }).catch(err => {
            this.$toastr.error("Connection to backend API service failed. Please manually refresh the webpage. If it still doesn't work, please restart the WebUI.<br/>连接到后台接口服务失败，请手动再刷新一次网页。如果还是不行，请重启WebUI。<br/><br/>" + err.message, globals.shortName, {timeOut: 20000})
            console.log(err)
        })
    },
    methods: {
        getLang(key) {
            return common.getLang(key, this.languageCode, this.languages)
        },
        init() {
            this.loadExtraNetworks()
            let dataListsKeys = ['languageCode', 'autoTranslate', 'autoTranslateToEnglish', 'autoTranslateToLocal', 'autoRemoveSpace', 'autoRemoveLastComma', 'autoKeepWeightZero', 'autoKeepWeightOne', 'autoBreakBeforeWrap', 'autoBreakAfterWrap', /*'hideDefaultInput', */'translateApi', 'enableTooltip', 'tagCompleteFile', 'onlyCsvOnAuto', 'extensionSelect.minimalist']
            this.prompts.forEach(item => {
                dataListsKeys.push(item.hideDefaultInputKey)
                dataListsKeys.push(item.hidePanelKey)
            })

            this.gradioAPI.getDatas(dataListsKeys).then(data => {
                if (data.languageCode !== null) {
                    let findLang = false
                    for (let key in this.languages) {
                        if (this.languages[key].code === data.languageCode) {
                            findLang = true
                            break
                        }
                    }
                    if (findLang) {
                        this.languageCode = data.languageCode
                        this.$forceUpdate()
                        this.gradioAPI.setData('languageCode', this.languageCode)
                    }
                } else {
                    let browserLang = navigator.language || navigator.userLanguage || ''
                    if (browserLang) {
                        for (let key in this.languages) {
                            if (common.isSameLang(this.languages[key].code, browserLang)) {
                                this.languageCode = this.languages[key].code
                                this.$forceUpdate()
                                this.gradioAPI.setData('languageCode', this.languageCode)
                                break
                            }
                        }
                    }
                }
                this.canOneTranslate = common.canOneTranslate(this.languageCode)
                if (data.autoTranslateToEnglish !== null) {
                    this.autoTranslateToEnglish = data.autoTranslateToEnglish
                }
                if (data.autoTranslateToLocal !== null) {
                    this.autoTranslateToLocal = data.autoTranslateToLocal
                }
                if (data.autoTranslate !== null) {
                    if (this.canOneTranslate) {
                        this.autoTranslate = data.autoTranslate
                        this.autoTranslateToEnglish = this.autoTranslate
                        this.autoTranslateToLocal = this.autoTranslate
                    } else {
                        this.autoTranslate = false
                    }
                } else {
                    if (this.canOneTranslate) {
                        this.autoTranslate = this.autoTranslateToEnglish || this.autoTranslateToLocal
                        this.autoTranslateToEnglish = true
                        this.autoTranslateToLocal = true
                    } else {
                        this.autoTranslate = false
                    }
                }
                if (data.autoRemoveSpace !== null) {
                    this.autoRemoveSpace = data.autoRemoveSpace
                }
                if (data.autoRemoveLastComma !== null) {
                    this.autoRemoveLastComma = data.autoRemoveLastComma
                }
                if (data.autoKeepWeightZero !== null) {
                    this.autoKeepWeightZero = data.autoKeepWeightZero
                }
                if (data.autoKeepWeightOne !== null) {
                    this.autoKeepWeightOne = data.autoKeepWeightOne
                }
                if (data.autoBreakBeforeWrap !== null) {
                    this.autoBreakBeforeWrap = data.autoBreakBeforeWrap
                }
                if (data.autoBreakAfterWrap !== null) {
                    this.autoBreakAfterWrap = data.autoBreakAfterWrap
                }
                /*if (data.hideDefaultInput !== null) {
                    this.hideDefaultInput = data.hideDefaultInput
                }*/
                if (data.enableTooltip !== null) {
                    this.enableTooltip = data.enableTooltip
                }
                localStorage.setItem('phystonPromptEnableTooltip', this.enableTooltip ? 'true' : 'false')
                this.updateTippyState()
                if (data.translateApi !== null) {
                    this.translateApi = data.translateApi
                    /*if (data.translateApi === 'alibaba_free') {
                        this.gradioAPI.setData('translateApi', this.translateApi)
                    } else {
                        this.translateApi = data.translateApi
                    }*/
                }
                if (data.tagCompleteFile !== null) {
                    this.tagCompleteFile = data.tagCompleteFile
                    this.$nextTick(() => {
                        this.$refs.translateSetting.getCSV(this.tagCompleteFile)
                    })
                } else {
                    /*if (typeof TAC_CFG === 'object' && typeof QUEUE_FILE_LOAD === 'object') {
                        QUEUE_FILE_LOAD.push(() => {
                            if (typeof TAC_CFG.translation !== 'object' || typeof TAC_CFG.translation.translationFile !== 'string') return
                            if (!TAC_CFG.translation.translationFile) return
                            this.tagCompleteFile = '\\extensions\\a1111-sd-webui-tagcomplete\\tags\\' + TAC_CFG.translation.translationFile
                            this.$refs.translateSetting.getCSV(this.tagCompleteFile)
                        })
                    }*/
                }
                if (data.onlyCsvOnAuto !== null) {
                    this.onlyCsvOnAuto = data.onlyCsvOnAuto
                }

                if (data['extensionSelect.minimalist'] === null) {
                    this.gradioAPI.setData('extensionSelect.minimalist', true)
                }

                this.updateTranslateApiConfig()
                this.$refs.extensionCss.init()

                this.prompts.forEach(item => {
                    if (data[item.hideDefaultInputKey] !== null) {
                        item.hideDefaultInput = data[item.hideDefaultInputKey]
                    }
                    if (data[item.hidePanelKey] !== null) {
                        item.hidePanel = data[item.hidePanelKey]
                    }
                    item.$prompt = common.gradioApp().querySelector("#" + item.prompt)
                    item.$textarea = item.$prompt.getElementsByTagName("textarea")[0]
                    item.$steps = common.gradioApp().querySelector("#" + item.steps)
                })
                this.$nextTick(() => {
                    this.prompts.forEach(item => {
                        const $prompt = common.gradioApp().querySelector("#" + item.id)
                        item.$prompt.parentElement.parentElement.parentElement.appendChild($prompt)
                        item.$prompt.parentElement.parentElement.style.display = item.hideDefaultInput ? 'none' : 'flex'
                        // item.$textarea.parentNode.appendChild($prompt)
                    })

                    this.startWatchSave = true
                })

                this.handlePaste()

                /*this.gradioAPI.getVersion().then(res => {
                    this.version = res.version
                    this.latestVersion = res.latest_version
                    this.isLatestVersion = res.version === res.latest_version
                })*/

                // todo: test
                // this.$refs.about.open()
                // this.$refs.chatgptPrompt.open()
                // this.$refs.promptFormat.open()
                // this.$refs.translateSetting.open(this.translateApi)
                /*this.onShowFavorite('phystonPrompt_txt2img_prompt', {
                    clientY: 150,
                    clientX: 283,
                })*/
            })
        },
        updateTippyState() {
            for (const $tippy of this.$tippyList) {
                if (this.enableTooltip) {
                    $tippy.enable()
                } else {
                    $tippy.disable()
                }
            }
        },
        updateTranslateApiConfig() {
            this.gradioAPI.getData('translate_api.' + this.translateApi).then(res => {
                let config = {}
                const apiItem = common.getTranslateApiItem(this.translateApis, this.translateApi)
                if (apiItem.config) {
                    for (const item of apiItem.config) {
                        if (apiItem.type === 'translators' && item.key === 'region' && !res['region']) {
                            config[item.key] = this.languageCode === 'zh_CN' || this.languageCode === 'zh_HK' || this.languageCode === 'zh_TW' ? 'China' : 'EN'
                        } else {
                            if (res) {
                                config[item.key] = res[item.key]
                            } else {
                                config[item.key] = item.default || ''
                            }
                        }
                    }
                    config['concurrent'] = apiItem.concurrent || 0
                }
                this.translateApiConfig = config
            })
        },
        onPromptFormatClick(e) {
            this.$refs.promptFormat.open(e)
        },
        onSelectLanguageClick(e) {
            this.$refs.selectLanguage.open(e)
        },
        onTranslateApiClick() {
            this.$refs.translateSetting.open(this.translateApi)
        },
        onSelectThemeClick() {
            this.$refs.extensionCss.open()
        },
        handlePaste() {
            if (typeof gradioApp !== 'function') return
            const $pastes = gradioApp().querySelectorAll("#paste")
            if (!$pastes || $pastes.length <= 0) return
            $pastes.forEach(($paste, index) => {
                // 拷贝一个新的按钮
                const $pasteNew = $paste.cloneNode(true)
                $pasteNew.id = 'paste-new-' + index
                $pasteNew.innerHTML = '🗒'
                // 加到原来的按钮后面一个
                $paste.parentNode.insertBefore($pasteNew, $paste.nextSibling)
                // 原来的按钮隐藏
                // $paste.style.display = 'none'
                // 监听新按钮点击事件
                $pasteNew.addEventListener('click', () => {
                    this.pasteBtn = $paste
                    this.openPastePopup()
                })
                this.pasteTitle = $paste.title
            });
        },
        openPastePopup() {
            this.pasteContent = ''
            this.pasteLoading = false
            this.showPastePopup = true
        },
        closePastePopup() {
            this.showPastePopup = false
        },
        onClickPasteSubmit() {
            this.pasteLoading = true
            const ele = get_uiCurrentTabContent()
            let $textarea = null
            let $textareaNeg = null
            let $prompt = null
            let $promptNeg = null
            let ids = []
            for (const item of this.prompts) {
                if (item.tab == ele.id) {
                    console.log(item)
                    ids.push(item.id)
                    if (item.neg) {
                        $textareaNeg = item.$textarea
                        $promptNeg = item.$prompt
                    } else {
                        $textarea = item.$textarea
                        $prompt = item.$prompt
                    }
                }
            }
            if (!$textarea || !$prompt || !$promptNeg) {
                this.pasteLoading = false
                return
            }
            $textarea.value = this.pasteContent
            $textarea.dispatchEvent(new Event('input'))
            this.pasteBtn.dispatchEvent(new Event('click'))

            setTimeout(() => {
                let interval = 0
                let intervalI = 0
                interval = setInterval(() => {
                    intervalI++
                    if (intervalI > 100) {
                        this.pasteLoading = false
                        clearInterval(interval)
                        return
                    }
                    if ($prompt.getElementsByClassName('hide').length > 0 && $promptNeg.getElementsByClassName('hide').length > 0) {
                        this.pasteLoading = false
                        this.closePastePopup()
                        clearInterval(interval)
                        common.hideCompleteResults($textarea)
                        common.hideCompleteResults($textareaNeg)
                        ids.forEach((id, index) => {
                            setTimeout(() => {
                                this.$refs[id][0].onTextareaChange(true)
                            }, 1000)
                        })
                    }
                }, 100)
            }, 1000)
        },
        onUpdateHideDefaultInput(id, value) {
            const item = this.prompts.find(item => item.id == id)
            if (!item) return
            item.hideDefaultInput = value
            this.gradioAPI.setData(item.hideDefaultInputKey, item.hideDefaultInput)
            item.$prompt.parentElement.parentElement.style.display = item.hideDefaultInput ? 'none' : 'flex'
        },
        onUpdateHidePanel(id, value) {
            const item = this.prompts.find(item => item.id == id)
            if (!item) return
            item.hidePanel = value
            this.gradioAPI.setData(item.hidePanelKey, item.hidePanel)
        },
        onShowHistory(id, e) {
            this.$refs.favorite.hide()
            this.historyCurrentPrompt = id
            const item = this.prompts.find(item => item.id == id)
            if (!item) return
            this.$refs.history.show(item.historyKey, e)
        },
        onUseHistory(history) {
            if (!this.historyCurrentPrompt) return
            const item = this.prompts.find(item => item.id == this.historyCurrentPrompt)
            if (!item) return
            this.$refs[item.id][0].useFavorite(history)
        },
        onShowFavorite(id, e) {
            this.$refs.history.hide()
            this.favoriteCurrentPrompt = id
            const item = this.prompts.find(item => item.id == id)
            if (!item) return
            this.$refs.favorite.show(item.favoriteKey, e)
        },
        onUseFavorite(favorite) {
            if (!this.favoriteCurrentPrompt) return
            const item = this.prompts.find(item => item.id == this.favoriteCurrentPrompt)
            if (!item) return
            this.$refs[item.id][0].useFavorite(favorite)
        },
        onRefreshFavorites(key) {
            this.$refs.favorite.getFavorites(key)
        },
        onShowChatgpt(id, e) {
            this.chatgptCurrentPrompt = id
            const item = this.prompts.find(item => item.id == id)
            if (!item) return
            this.$refs.chatgptPrompt.open()
        },
        onUseChatgpt(prompt) {
            if (!this.chatgptCurrentPrompt) return
            const item = this.prompts.find(item => item.id == this.chatgptCurrentPrompt)
            if (!item) return
            this.$refs[item.id][0].useChatgpt(prompt)
        },
        onShowAbout() {
            this.$refs.about.open()
        },
        onSwitchTheme() {
            /*if (common.gradioApp().classList.contains(this.theme)) {
                common.gradioApp().classList.remove(this.theme)
            }*/
            this.theme = this.theme === 'dark' ? 'light' : 'dark'
            // 判断当前 url 是否有参数 __theme，如果有则替换，没有则添加
            let currentUrl = window.location.href
            let url = new URL(currentUrl)
            let params = new URLSearchParams(url.search)
            if (params.has('__theme')) {
                params.set('__theme', this.theme);
            } else {
                params.append('__theme', this.theme);
            }
            let newUrl = url.origin + url.pathname + '?' + params.toString()
            /*if (!common.gradioApp().classList.contains(this.theme)) {
                common.gradioApp().classList.add(this.theme)
            }*/
            window.location.href = newUrl
        },
    },
}
</script>
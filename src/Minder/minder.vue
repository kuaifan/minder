<template>
    <div :id="id" class="container"></div>
</template>

<style scoped>
.container {
    font-family: Arial, Hiragino Sans GB, Microsoft YaHei, WenQuanYi Micro Hei, sans-serif;
}
</style>
<script>
import {generateMixed} from 'vue-kityminder-ggg/src/utils/index.js';
import 'vue-kityminder-ggg/examples/styles/minder.css';
import JSPDF from 'jspdf';

export default {
    name: 'mind-editor',
    props: {
        value: {
            type: Object,
            default: function () {
                return {}
            }
        },
        AccessKey: {
            type: String,
            default: ''
        },
        SecretKey: {
            type: String,
            default: ''
        },
        Domain: {
            type: String,
            default: ''
        },
        scope: {
            type: String,
            default: ''
        },
        isImageUpload: {
            type: Boolean,
            default: true
        },
        saveShow: {
            type: Boolean,
            default: true
        },
        readOnly: {
            type: Boolean,
            default: false
        },
        id: {
            type: String,
            default: 'minder-component-' + generateMixed(12)
        },
    },
    data() {
        return {
            minder: null,
            isHand: false,
            bakValue: '',

            zoom: 100,
        };
    },
    methods: {
        execCommand(var1, var2) {
            if (this.readOnly === true) {
                this.minder.enable();
                this.$nextTick(() => {
                    this.minder.execCommand(var1, var2);
                    this.$nextTick(() => {
                        this.minder.disable();
                        if (this.isHand) {
                            this.minder.execCommand('Hand');
                        }
                    });
                });
            } else {
                this.minder.execCommand(var1, var2);
            }
        },
        exportHandle(n, filename) {
            filename = filename || (this.value.root.data.text || 'Untitled');
            if (n === 0 || n === 'png') {
                this.minder.exportData('png').then((content) => {
                    let element = document.createElement('a');
                    element.setAttribute('href', content);
                    element.setAttribute('download', filename);
                    element.style.display = 'none';
                    document.body.appendChild(element);
                    element.click();
                    document.body.removeChild(element);
                });
            } else if (n === 1 || n === 'pdf') {
                this.minder.exportData('png').then((content) => {
                    let doc = new JSPDF();
                    doc.addImage(content, 'PNG', 0, 0, 0, 0);
                    doc.save(`${filename}.pdf`);
                });
            }
        },
        rendData() {
            this.$nextTick(() => {
                setTimeout(() => {
                    if (this.minder !== null) {
                        if (this.bakValue == JSON.stringify(this.value)) {
                            return;
                        }
                        this.bakValue = JSON.stringify(this.value);
                        this.minder.importJson(this.value);
                        return;
                    }
                    window.__minderReadOnly = this.readOnly;
                    const Editor = require('./editor');
                    this.minder = window.editor = new Editor(document.getElementById(this.id)).minder;
                    this.bakValue = JSON.stringify(this.value);
                    this.minder.importJson(this.value);
                    if (this.readOnly === true) {
                        this.minder.disable();
                        this.minder.execCommand('Hand');
                        this.isHand = true;
                    }
                    this.$emit('minderHandle', this.minder);
                    this.minder.on('contentchange', e => {
                        const newJson = this.minder.exportJson();
                        if (this.bakValue == JSON.stringify(newJson)) {
                            return;
                        }
                        this.bakValue = JSON.stringify(newJson);
                        this.$emit('input', newJson);
                    });
                }, 300)
            });
        }
    },
    watch: {
        value: {
            handler: function (newObj) {
                if (typeof newObj !== "object" || newObj === null) {
                    newObj = {
                        root: newObj,
                        theme: "fresh-blue",
                        template: "default",
                    };
                }
                if (typeof newObj.root !== "object" || newObj.root === null || newObj.root.length == 0) {
                    newObj.root = {
                        data: {
                            id: generateMixed(12),
                            text: 'Default',
                        },
                        children: []
                    }
                }
                if (typeof newObj.theme !== "string") {
                    newObj.theme = "fresh-blue";
                }
                if (typeof newObj.template !== "string") {
                    newObj.template = "default";
                }
                this.rendData();
            },
            deep: true,
            immediate: true
        },
        zoom(val) {
            this.execCommand('Zoom', val)
        }
    }
};
</script>
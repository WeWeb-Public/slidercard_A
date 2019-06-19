<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div class="slidercard" :style="customStyle">
        <!-- wwManager:start -->
        <wwSectionEditMenu :sectionCtrl="sectionCtrl" :options="openOptions"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <wwObject class="background" :ww-object="section.data.bg" ww-category="background"></wwObject>
        <!--TOP WWOBJS-->
        <div class="top-ww-objs">
            <wwLayoutColumn
                tag="div"
                ww-default="ww-image"
                :ww-list="section.data.topWwObjs"
                class="top-ww-obj"
                @ww-add="add(section.data.topWwObjs, $event)"
                @ww-remove="remove(section.data.topWwObjs, $event)"
            >
                <wwObject v-for="topWwObj in section.data.topWwObjs" :key="topWwObj.uniqueId" :ww-object="topWwObj"></wwObject>
            </wwLayoutColumn>
        </div>

        <div class="container">
            <div class="card">
                <wwObject class="card-slider-background" :ww-object="section.data.containerBackground" ww-category="background"></wwObject>
                <wwLayoutColumn
                    tag="div"
                    ww-default="ww-text"
                    :ww-list="section.data.titles"
                    class="content card-title"
                    @ww-add="add(section.data.titles, $event)"
                    @ww-remove="remove(section.data.titles, $event)"
                >
                    <wwObject tag="div" v-for="title in section.data.titles" :key="title.uniqueId" :ww-object="title"></wwObject>
                </wwLayoutColumn>

                <v-touch ref="swiper" :enabled="!editMode" @swipeleft="slide('right')" @swiperight="slide('left')" :swipe-options="{ direction: 'horizontal', threshold: 10, velocity: 0.2 }">
                    <div class="container-center" :style="[getTotalWidth]">
                        <div class="thumbnail-container" v-for="(feature, index) in section.data.features" :key="feature.uniqueId" :style="[getColWidth]">
                            <!-- wwManager:start -->
                            <wwContextMenu
                                tag="div"
                                class="contextmenu"
                                v-if="editMode"
                                @ww-add-before="addFeature(index, 'before')"
                                @ww-add-after="addFeature(index, 'after')"
                                @ww-remove="removeFeature(index)"
                            >
                                <div class="wwi wwi-config"></div>
                            </wwContextMenu>
                            <!-- wwManager:end -->
                            <wwObject class="background" :ww-object="feature.background" ww-category="background"></wwObject>

                            <wwLayoutColumn
                                tag="div"
                                ww-default="ww-image"
                                :ww-list="feature.contents"
                                class="content"
                                @ww-add="add(feature.contents, $event)"
                                @ww-remove="remove(feature.contents, $event)"
                            >
                                <wwObject tag="div" v-for="content in feature.contents" :key="content.uniqueId" :ww-object="content"></wwObject>
                            </wwLayoutColumn>
                        </div>
                    </div>
                </v-touch>
            </div>
            <div v-if="canSwipeLeft " class="prev-button">
                <wwObject tag="div" :ww-object="section.data.prevButton" @click="slide('left')"></wwObject>
            </div>
            <div v-if="canSwipeRight" class="next-button">
                <wwObject tag="div" :ww-object="section.data.nextButton" @click="slide('right')"></wwObject>
            </div>
        </div>

        <!--BOTTOM WWOBJS-->
        <div class="bottom-ww-objs">
            <wwLayoutColumn
                tag="div"
                ww-default="ww-image"
                :ww-list="section.data.bottomWwObjs"
                class="top-ww-obj"
                @ww-add="add(section.data.bottomWwObjs, $event)"
                @ww-remove="remove(section.data.bottomWwObjs, $event)"
            >
                <wwObject v-for="bottomWwObj in section.data.bottomWwObjs" :key="bottomWwObj.uniqueId" :ww-object="bottomWwObj"></wwObject>
            </wwLayoutColumn>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>

const VueTouch = require('vue-touch')

Vue.use(VueTouch, { name: 'v-touch' })
import { getNewFeature } from "./defaultFeature"

export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            maxThumbnailsPerLine: 3,
            colsPosition: 0,
        }
    },
    computed: {
        //Get the section object here !
        // It contains all the info and data about the section
        // Use it has you like !
        section() {
            return this.sectionCtrl.get();
        },
        editMode() {
            return this.sectionCtrl.getEditMode() == 'CONTENT'
        },
        featuresLength() {
            return this.section.data.features.length
        },
        getColWidth() {
            let value = 100 / this.featuresLength
            return { 'width': 'calc(' + value + '%)' }
        },
        getTotalWidth() {
            let value = 100 * this.featuresLength / this.maxThumbnailsPerLine;
            return { 'width': 'calc(' + value + '%)', 'transform': 'translateX(-' + this.colsPosition + '%)' }
        },
        canSwipeRight() {
            return this.colsPosition < (100 - (100 / this.featuresLength * this.maxThumbnailsPerLine))
        },
        canSwipeLeft() {
            return this.colsPosition > 0 //fix
        },
        customStyle() {
            return {
                '--cardBorderRadius': this.section.data.cardBorderRadius + 'px',
                '--shadowColor': this.section.data.shadowColor,
                '--shadowColorAfter': this.section.data.shadowColorAfter
            }
        }

    },

    created() {

        let needUpdate = false
        //Initialize section data
        this.section.data = this.section.data || {};

        if (!this.section.data.bg) {
            this.section.data.bg = wwLib.wwObject.getDefault({
                type: 'ww-color'
            });
            needUpdate = true
        }
        if (!this.section.data.containerBackground) {
            this.section.data.containerBackground = wwLib.wwObject.getDefault({
                type: 'ww-color'
            });
            needUpdate = true
        }
        if (!this.section.data.nextButton) {
            this.section.data.nextButton = wwLib.wwObject.getDefault({
                type: 'ww-icon'
            });
            needUpdate = true
        }
        if (!this.section.data.prevButton) {
            this.section.data.prevButton = wwLib.wwObject.getDefault({
                type: 'ww-icon'
            });
            needUpdate = true
        }

        if (!this.section.data.features) {
            this.section.data.features = []
            needUpdate = true
        }
        if (!this.section.data.titles) {
            this.section.data.titles = []
            needUpdate = true
        }

        if (_.isEmpty(this.section.data.features)) {
            for (let i = 0; i < 3; i++) {
                this.section.data.features.push(
                    {
                        background: wwLib.wwObject.getDefault({ type: 'ww-color', data: { color: 'white' } }),
                        contents: []
                    },
                )
            }

            needUpdate = true;
        }

        if (_.isEmpty(this.section.data.topWwObjs)) {
            this.section.data.topWwObjs = [];
            needUpdate = true;
        }
        if (_.isEmpty(this.section.data.bottomWwObjs)) {
            this.section.data.bottomWwObjs = [];
            needUpdate = true;
        }

        if (needUpdate) {
            this.sectionCtrl.update(this.section);
        }
    },
    mounted() {
        this.init();
    },
    beforeDestroy() {
        window.removeEventListener("resize", this.setThumbnailsPerLine);
    },
    methods: {
        init() {
            this.setThumbnailsPerLine();
            window.addEventListener("resize", this.setThumbnailsPerLine);
        },
        setThumbnailsPerLine() {
            try {
                let width = window.innerWidth;
                if (width < 576) {
                    this.maxThumbnailsPerLine = 1;
                }
                else if (width < 992) {
                    this.maxThumbnailsPerLine = 2;
                }
                else {
                    this.maxThumbnailsPerLine = 3;
                }

            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        slide(direction) {
            if (!this.editMode) {
                if (direction === 'right' && this.canSwipeRight) {
                    if (this.colsPosition < 100 * (this.featuresLength - 1))
                        this.colsPosition += 100 / this.featuresLength;
                } else if (direction === 'right' && !this.canSwipeRight) {
                    this.colsPosition = 0;
                }
                else {
                    if (this.colsPosition > 0)
                        this.colsPosition -= 100 / this.featuresLength;
                }
                if (this.colsPosition < 1) {
                    this.colsPosition = 0;
                }
            }

        },

        /* wwManager:start */
        add(list, options) {
            try {
                list.splice(options.index, 0, options.wwObject);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        /* add a new section to the slider */
        addFeature(_index, where) {
            try {
                const up = (where == 'after') ? parseInt(1) : 0;
                const index = _index + up
                const newCard = getNewFeature()

                this.section.data.features.splice(index, 0, newCard);
                this.sectionCtrl.update(this.section);
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        removeFeature(index) {
            try {
                this.section.data.features.splice(index, 1);
                if (!this.section.data.features.length) {
                    this.addFeature(0, 'after');
                }
                this.sectionCtrl.update(this.section);
                this.slide('left')
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);

            }
        },

        async openOptions() {
            try {

                wwLib.wwPopups.addStory('WWSLIDER_CUSTOM', {
                    title: {
                        en: 'Cards config',
                        fr: 'Configuration des cartes'
                    },
                    type: 'wwPopupForm',
                    storyData: {
                        fields: [
                            {
                                label: {
                                    en: 'Shadow Color:',
                                    fr: 'Couleur de l\'ombre :'
                                },
                                type: 'text',
                                key: 'shadowColor',
                                valueData: 'shadowColor',
                                desc: {
                                    en: 'Example: 0 10px 40px 0 rgba(113, 124, 137, 0.2)',
                                    fr: 'Exemple : 0 10px 40px 0 rgba(113, 124, 137, 0.2)'
                                }
                            },
                            {
                                label: {
                                    en: 'Shadow Color After hover:',
                                    fr: 'Couleur de l\'ombre après le hover:'
                                },
                                type: 'text',
                                key: 'shadowColorAfter',
                                valueData: 'shadowColorAfter',
                                desc: {
                                    en: 'Example: 0 16px 32px rgba(113, 124, 137, 0.4)',
                                    fr: 'Exemple : 0 16px 32px rgba(113, 124, 137, 0.4)'
                                }
                            },
                            {
                                label: {
                                    en: 'Border radius in px:',
                                    fr: 'Arrondis des coins en px :'
                                },
                                type: 'text',
                                key: 'cardBorderRadius',
                                valueData: 'cardBorderRadius',
                                desc: {
                                    en: 'Edit card border radius',
                                    fr: 'Changer la bordure des coins des cartes'
                                }
                            },
                        ]
                    },
                    buttons: {
                        NEXT: {
                            text: {
                                en: 'Finish',
                                fr: 'Terminer'
                            },
                            next: null
                        }
                    }
                })
                let options = {
                    firstPage: 'WWSLIDER_CUSTOM',
                    data: {
                        section: this.section,
                        shadowColor: this.section.data.shadowColor,
                        shadowColorAfter: this.section.data.shadowColorAfter,
                        cardBorderRadius: this.section.data.cardBorderRadius
                    },
                }

                const result = await wwLib.wwPopups.open(options)
                let updateSection = false;
                if (typeof (result) != 'undefined') {

                    if (typeof (result.shadowColorAfter) != 'undefined') {
                        this.section.data.shadowColor = result.shadowColor;
                        updateSection = true;
                    }
                    if (typeof (result.shadowColorAfter) != 'undefined') {
                        this.section.data.shadowColorAfter = result.shadowColorAfter;
                        updateSection = true;
                    }
                    if (typeof (result.cardBorderRadius) != 'undefined') {
                        this.section.data.cardBorderRadius = result.cardBorderRadius;
                        updateSection = true;
                    }

                    if (updateSection) {
                        this.sectionCtrl.update(this.section);
                    }
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        remove(list, options) {
            try {
                list.splice(options.index, 1);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }

        },

        /* wwManager:end */
    }
};
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style lang='scss' scoped>
.slidercard {
    position: relative;
    .background {
        position: absolute;
        top: 0;
        left: 0;
        height: 100%;
        width: 100%;
    }

    .top-ww-objs {
        position: relative;
        padding-top: 20px;
        margin-bottom: 20px;
        .top-ww-obj {
            position: relative;
        }
    }

    .bottom-ww-objs {
        position: relative;
        padding-bottom: 20px;
        margin-top: 20px;
        .bottom-ww-obj {
            position: relative;
        }
    }

    .container {
        position: relative;
        width: 90%;
        border-radius: var(--cardBorderRadius);
        box-shadow: var(--shadowColor);
        &:hover {
            box-shadow: var(--shadowColorAfter);
        }

        @media (min-width: 768px) {
            width: 80%;
        }

        @media (min-width: 1200px) {
            width: 80%;
        }
        .prev-button {
            position: absolute;
            left: 0;
            top: 50%;
            transform: translate(-50%, -50%);
            cursor: pointer;
            z-index: 10;
        }
        .next-button {
            display: block;
            position: absolute;
            right: 0;
            top: 50%;
            transform: translate(50%, -50%);
            cursor: pointer;
            z-index: 10;
        }
        .card {
            overflow-x: hidden;
            position: relative;
            border-radius: var(--cardBorderRadius);
            .card-slider-background {
                position: absolute;
                top: 0;
                left: 0;
                height: 100%;
                width: 100%;
            }
            .card-title {
                position: relative;
                width: 100%;
                margin-top: 20px;
            }
            .container-center {
                display: flex;
                transition: transform 0.4s ease;
                @media (min-width: 1024px) {
                    justify-content: center;
                    flex-wrap: wrap;
                }

                .thumbnail-container {
                    position: relative;
                    background-color: white;
                    min-height: 100px;
                    overflow: hidden;
                    .background {
                        overflow: hidden;
                    }
                    .content {
                        position: relative;
                    }
                    /* wwManager:start */

                    .contextmenu {
                        position: absolute;
                        top: 0;
                        left: 0;
                        width: 30px;
                        height: 30px;
                        color: white;
                        background-color: #ef811a;
                        border-radius: 100%;
                        display: flex;
                        justify-content: center;
                        align-items: center;
                        font-size: 1.2rem;
                        cursor: pointer;
                        z-index: 2;
                    }
                    .contextmenu-center {
                        left: calc(50% - 15px);
                    }
                    /* wwManager:end */
                }
            }
        }
    }
}

/* wwManager:start */
.ww-editing {
    .container {
        &:hover {
            transform: none !important;
        }
    }
}
/* wwManager:end */
</style>

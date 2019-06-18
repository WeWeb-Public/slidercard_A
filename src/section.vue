<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div class="slidercard">
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

        <div class="container hidden-mobile">
            <div class="prev-button">
                <wwObject tag="div" :ww-object="section.data.nextButton" @click="slide('left')"></wwObject>
            </div>

            <div class="card">
                <div class="container-center" :style="[getTotalWidth]">
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
            </div>
            <div class="next-button">
                <wwObject tag="div" :ww-object="section.data.prevButton" @click="slide('right')"></wwObject>
            </div>
        </div>

        <v-touch
            ref="swiper"
            :enabled="!editMode"
            @swipeleft="nextSlide()"
            @swiperight="prevSlide()"
            :swipe-options="{ direction: 'horizontal', threshold: 10, velocity: 0.2 }"
            class="container mobile-wrapper"
        >
            <div class="container-center" :style="[mobileStyle, mobileTransition]">
                <div class="thumbnail-container" v-for="feature in section.data.features" :key="feature.uniqueId" :style="cardWidth">
                    <!-- wwManager:start -->
                    <wwContextMenu
                        tag="div"
                        class="contextmenu contextmenu-center"
                        v-if="editMode"
                        @ww-add-before="addFeature(index, 'before')"
                        @ww-add-after="addFeature(index, 'after')"
                        @ww-remove="removeFeature(index)"
                    >
                        <div class="wwi wwi-config"></div>
                    </wwContextMenu>
                    <!-- wwManager:end -->
                    <wwObject class="background" :ww-object="feature.background" ww-category="background"></wwObject>
                    <!-- feature -->
                    <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="feature.contents" class="content" @ww-add="add(feature.contents, $event)" @ww-remove="remove(feature.contents, $event)">
                        <wwObject tag="div" v-for="content in feature.contents" :key="content.uniqueId" :ww-object="content"></wwObject>
                    </wwLayoutColumn>
                </div>
            </div>
            <div class="content-dots-wrapper">
                <li
                    v-for="(dot, index) in section.data.features"
                    class="content-dot"
                    :style="{'background': ((sliderPosition == index) ? section.data.dotColor : ''), 'border-color': section.data.dotColor}"
                    :key="dot.uniqueId"
                >
                    <div class="dot" @click="switchToIndex(sliderPosition, index)"></div>
                </li>
            </div>
        </v-touch>

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
            columnWidth: { 'width': 'calc(33.33% - 30px)' },
            sliderPosition: 0,
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

        mobileStyle() {
            return { 'width': 'calc(' + this.featuresLength + ' * 100%)' }
        },
        laptopStyle() {
            return { 'width': 'calc(' + this.featuresLength + ' * 100%)' }
        },
        mobileTransition() {
            return { 'transform': 'translate(calc(' + (- this.sliderPosition * (100 / this.featuresLength)) + '% + ' + this.sliderPosition * 32 + 'px ), 0)' }
        },
        cardWidth() {
            return { 'width': 'calc(' + (100 / this.featuresLength) + '% - 50px)' }
        }
    },

    created() {

        let needUpdate = false
        //Initialize section data
        this.section.data = this.section.data || {};
        this.section.data.dotColor = this.section.data.dotColor || "#9013FE"

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
        if (!this.section.data.thumbnailsPerLine) {
            this.section.data.thumbnailsPerLine = 4;
            needUpdate = true;
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
        nextSlide() {
            try {
                this.sliderPosition++
                if (this.sliderPosition > this.featuresLength - 1)
                    this.sliderPosition = 0
                this.$forceUpdate()
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        prevSlide() {
            try {
                this.sliderPosition--
                if (this.sliderPosition < 0)
                    this.sliderPosition = this.featuresLength - 1
                this.$forceUpdate()
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }

        },

        switchToIndex(index, position) {
            try {
                if (position < this.section.data.features.length && index != position) {
                    this.sliderPosition = position
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);

            }

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


                switch (Math.min(this.section.data.thumbnailsPerLine, this.maxThumbnailsPerLine)) {
                    case 1:
                        this.columnWidth = { 'width': "calc(100% - 30px)" };
                        break;
                    case 2:
                        this.columnWidth = { 'width': "calc(50% - 30px)" };
                        break;
                    default:
                        this.columnWidth = { 'width': "calc(30% - 30px)" };
                        break;
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        slide(direction) {
            console.log('direction:', direction)
            console.log('this.featuresLength :', this.featuresLength)
            console.log('this.colsPosition:', this.colsPosition)
            if (direction === 'right') {
                if (this.colsPosition >= 100 * (this.featuresLength - 1)) { return; }

                this.colsPosition += 100 / this.featuresLength;
            } else {
                if (this.colsPosition <= 0) { return; }
                this.colsPosition -= 100 / this.featuresLength;
            }
            if (this.colsPosition < 1) {
                this.colsPosition = 0;
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
                this.prevSlide()
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);

            }
        },

        /* add picture */

        addElement(list, _index, where) {
            try {
                const up = (where == 'after') ? parseInt(1) : 0;
                const index = _index + up
                let newCopie = JSON.parse(JSON.stringify(list[0]))

                wwLib.wwUtils.changeUniqueIds(newCopie)
                list.splice(index, 0, newCopie);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        removeElement(list, index) {
            try {
                if (list.length > 1) {
                    list.splice(index, 1);
                    this.sectionCtrl.update(this.section);
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        async openOptions() {
            try {

                wwLib.wwPopups.addStory('WWSLIDER_CUSTOM', {
                    title: {
                        en: 'Fill in code',
                        fr: 'Inserer le code'
                    },
                    type: 'wwPopupForm',
                    storyData: {
                        fields: [
                            {
                                label: {
                                    en: 'Column per line',
                                    fr: 'Nombre de colonnes par ligne :'
                                },
                                type: 'text',
                                key: 'columnPerLine',
                                valueData: 'section.data.thumbnailsPerLine',
                                desc: {
                                    en: 'The number of column per line',
                                    fr: 'Le nombre de colonnes par ligne'
                                }

                            },
                            {
                                label: {
                                    en: 'Navigation dots color:',
                                    fr: 'Couleur des points de navigations :'
                                },
                                type: 'color',
                                key: 'dotsColor',
                                valueData: 'section.data.dotColor',
                                desc: {
                                    en: 'Choose a Nav dots color:',
                                    fr: 'Changer la couleur des points de navigations :'
                                },
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
                        columnPerLine: this.section.data.thumbnailsPerLine,
                        section: this.section,

                    },
                }

                const result = await wwLib.wwPopups.open(options)
                if (typeof (result) != 'undefined') {
                    if (typeof (result.dotsColor) != 'undefined') {
                        this.section.data.dotColor = result.dotsColor
                    }
                    if (result.columnPerLine) {
                        this.section.data.thumbnailsPerLine = result.columnPerLine;
                        this.sectionCtrl.update(this.section);
                        this.setThumbnailsPerLine();
                    }
                    this.sectionCtrl.update(this.section);
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        /* Used on mobile version to swipe */
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

    .top-ww-objs,
    .bottom-ww-objs {
        position: relative;
        margin: 10px 0;
        .top-ww-obj,
        .bottom-ww-obj {
            position: relative;
        }
    }

    .container {
        position: relative;
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
            position: absolute;
            right: 0;
            top: 50%;
            transform: translate(calc(-50% + 40px), -50%);
            cursor: pointer;
            z-index: 10;
        }
        .card {
            overflow-x: hidden;
            position: relative;
            .container-center {
                display: flex;
                transition: transform 0.5s ease;
                @media (min-width: 1024px) {
                    justify-content: center;
                    flex-wrap: wrap;
                }
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
                .thumbnail-container {
                    position: relative;
                    background-color: white;
                    min-height: 100px;
                    overflow: hidden;
                    .background {
                        border-radius: 7px;
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

    .content-dots-wrapper {
        display: flex;
        list-style: none;
        justify-content: center;
        position: relative;
        padding-bottom: 15px;
        margin-top: 20px;

        .content-dot {
            margin-right: 15px;
            border-radius: 100%;
            border-width: 2px;
            border-style: solid;
            .dot {
                cursor: pointer;
                width: 15px;
                height: 15px;
                pointer-events: all;
            }
        }
    }
    .hidden-mobile {
        display: none;
        @media (min-width: 1024px) {
            display: block;
        }
    }
    .mobile-wrapper {
        display: block;
        position: relative;
        @media (min-width: 1024px) {
            display: none;
        }
    }
}
</style>

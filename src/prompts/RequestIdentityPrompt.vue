<template>
    <section class="prompt-body">

        <section class="prompt-actions" v-if="filteredIdentities().length">
            <btn :text="locale(langKeys.BUTTON_Deny)" half="true" v-on:clicked="denied"></btn>
            <btn :text="locale(langKeys.BUTTON_Accept)" half="true" is-blue="true" v-on:clicked="accepted"></btn>
        </section>

        <section class="prompt-actions" v-else>
            <btn :text="locale(langKeys.BUTTON_Cancel)" v-on:clicked="denied"></btn>
        </section>

        <section class="partitioned">

            <section class="partition">

                <section v-if="identityFields.length">
                    <section class="description">
                        <b>{{prompt.domain}}</b> {{locale(langKeys.REQUEST_Identity)[0]}}
                    </section>

                    <section class="key-value">
                        <figure class="key">{{locale(langKeys.GENERIC_Requires)}}</figure>
                        <figure class="value">
                            {{identityFields.join(', ')}}
                        </figure>
                    </section>

                    <section class="description">{{locale(langKeys.REQUEST_Identity)[1]}}</section>
                    <section class="description">{{locale(langKeys.REQUEST_Identity)[2]}}</section>
                    <section class="description"><b>{{locale(langKeys.REQUEST_Identity)[3]}}</b></section>
                </section>

                <section v-else>
                    <section class="description"><b>{{prompt.domain}}</b> {{locale(langKeys.REQUEST_Identity)[3]}}</section>
                </section>

            </section>

            <search v-on:changed="changed => bind(changed, 'searchText')" v-if="filteredIdentities().length"></search>

            <section class="partition scroller" v-if="filteredIdentities().length">


                <section v-for="identity in filteredIdentities()" class="panel-box" :class="{'disabled':identity.disabled}">

                    <!-- Header -->
                    <section class="panel">
                        <figure class="header big identity-header">{{identity.name}}</figure>
                        <figure v-if="!identity.disabled" class="select-identity"
                                v-on:click="selectIdentity(identity)"
                                :class="{'selected':selectedIdentity && selectedIdentity.publicKey === identity.publicKey}">
                            {{locale(langKeys.BUTTON_SelectIdentity)}}
                        </figure>
                        <!--<figure class="header small margin"><i class="fa fa-globe"></i>{{identity.network.host}}</figure>-->
                    </section>

                    <!-- Matching Requirements / Properties -->
                    <section class="panel">
                        <figure class="header small reverse-margin">{{locale(langKeys.GENERIC_RequiredProperties)}}</figure>
                        <section class="items">
                            <section class="item" v-for="prop in identityFields">
                                <figure>
                                    <span>{{prop}}</span>
                                    <span>{{formatPropValue(prop, identity.getPropertyValueByName(prop))}}</span>
                                </figure>
                            </section>
                        </section>
                    </section>

                </section>

            </section>

            <section class="partition" v-else>
                <section class="nothing-here">
                    <figure class="header"><b>{{locale(langKeys.REQUEST_IdentityNoIdentities)[0]}}</b></figure>
                    <figure class="sub-header">{{locale(langKeys.REQUEST_IdentityNoIdentities)[1]}}</figure>
                </section>
            </section>

        </section>

    </section>
</template>

<script>
    import { mapActions, mapGetters, mapState } from 'vuex'
    import * as Actions from '../store/constants';
    import {RouteNames} from '../vue/Routing'
    import AlertMsg from '../models/alerts/AlertMsg'
    import IdentityService from '../services/IdentityService'
    import NotificationService from '../services/NotificationService'
    import Identity from '../models/Identity'

    export default {
        data(){ return {
            searchText:'',
            selectedIdentity:null,
        }},
        computed: {
            ...mapState([
                'scatter',
                'prompt'
            ]),
            ...mapGetters([
                'identities',
                'identityFields'
            ])
        },
        methods: {
            bind(changed, original) { this[original] = changed },
            filteredIdentities(){
                return this.identities
                    .filter(id => Object.keys(id.accounts).indexOf(this.prompt.network.unique()) > -1)
                    .filter(id => id.hasRequiredFields(this.identityFields, this.prompt.network))
                    .filter(id => JSON.stringify(id).indexOf(this.searchText) !== -1)
                    .sort((a,b) => !a.disabled || !b.disabled ? 1 : -1)
            },
            formatPropValue(prop, propValue){
                switch(prop){
//                    case 'account': return `${propValue.name}@${propValue.authority}`;
                    case 'country': return propValue.name;
                }

                return propValue;
            },
            selectIdentity(identity){
                this.selectedIdentity = identity;
            },
            accepted(){
                if(!this.selectedIdentity){
                    this[Actions.PUSH_ALERT](AlertMsg.YouMustSelectAnIdentity());
                    return false;
                }
                const identity = this.identities.find(id => id.publicKey === this.selectedIdentity.publicKey);
                this.prompt.responder(identity);
                NotificationService.close();
            },
            denied(){
                this.prompt.responder(null);
                NotificationService.close();
            },
            ...mapActions([
                Actions.UPDATE_STORED_SCATTER,
                Actions.PUSH_ALERT,
                Actions.PUSH_PROMPT
            ])
        }
    }
</script>

<style lang="scss">
    span {

        .red {

        }
    }
    .prompt-body {
        font-family: 'Open Sans', sans-serif;

        .identity-header {
            width:calc(100% - 98px);
            display:inline-block;
        }

        .select-identity {
            cursor: pointer;
            padding:0 10px;
            height:24px;
            line-height:21px;
            display:inline-block;
            color:#bebebe;
            font-family: 'Open Sans', sans-serif;
            font-size: 11px;
            border:1px solid #dfdfdf;
            border-radius: 4px;
            transition:all 0.2s ease;
            transition-property: background, border, color;

            &:hover, &.selected {
                background:#54a7fc;
                border:1px solid #54a7fc;
                color:#fff;
            }
        }

        .description {
            margin-top:5px;
            font-size:9px;
            color:#b8b8b8;

            b { color:#707070; }
        }

        .partitioned {
            overflow:hidden;
            height:436px;
            width:100%;

            .partition {
                width:50%;
                float:left;
                height:436px;
                overflow-y:auto;
                position: relative;
                display:block;

                &.scroller {
                    height:405px;
                }

                &:first-child {
                    /*padding:20px;*/
                    padding:40px 50px;

                    .key-value {
                        padding:20px;
                        border:2px dashed rgba(0,0,0,0.05);
                        border-radius: 4px;
                        margin-bottom:20px;
                        margin-top:20px;

                        .key {
                            font-size:9px;
                            color:#bbbbbb;
                        }

                        .value {
                            font-size:12px;
                            color:#707070;
                            font-weight:900;
                            font-style: italic;
                        }
                    }
                }

                &:last-child {
                    background:#f5f5f5;
                    padding:20px;
                }
            }
        }
    }
</style>
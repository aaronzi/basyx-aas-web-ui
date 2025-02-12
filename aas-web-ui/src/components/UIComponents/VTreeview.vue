<template>
    <div class="VTreeview">
        <!-- List Item with a Submodel / SubmodelelementCollection / submodelElement (like Property) -->
        <!-- TODO: Fix weird Ripple effect on isActive change to false -->
        <v-lazy transition="fade-transition">
            <v-list-item
                :style="{ 'padding-left': depth * 22 + 'px' }"
                density="compact"
                class="py-0"
                nav
                color="primary"
                :active="item.isActive"
                @click="toggleNode()">
                <v-list-item-title>{{ nameToDisplay(item) }}</v-list-item-title>
                <template #prepend>
                    <!-- Button to show/hide children -->
                    <v-btn
                        v-if="item.children"
                        size="small"
                        variant="plain"
                        :icon="showChildren ? 'mdi-menu-down' : 'mdi-menu-right'"
                        :ripple="false"
                        @click.stop="toggleChildren()"></v-btn>
                    <div v-else style="width: 40px; height: 40px"></div>
                    <!-- Lock Icon for Authorization Errors -->
                    <v-icon v-if="item.authorizationError" color="error"> mdi-folder-lock </v-icon>
                    <!-- Empty Submodel Icon -->
                    <v-icon v-else-if="item.modelType === 'Submodel' && !item.children" color="primary">
                        mdi-folder-alert
                    </v-icon>
                    <!-- Icon for Submodel with children (open/closed) -->
                    <v-icon v-else-if="item.modelType === 'Submodel' && item.children" color="primary">
                        {{ showChildren ? 'mdi-folder-open' : 'mdi-folder' }}
                    </v-icon>
                    <!-- Icon for empty SubmodelelementCollection -->
                    <v-icon
                        v-else-if="item.modelType === 'SubmodelElementCollection' && !item.children"
                        color="primary">
                        mdi-file-alert
                    </v-icon>
                    <!-- Icon for SubmodelelementCollection -->
                    <v-icon v-else-if="item.modelType === 'SubmodelElementCollection'" color="primary">
                        mdi-file-multiple
                    </v-icon>
                    <!-- Icon for empty SubmodelelementList -->
                    <v-icon v-else-if="item.modelType === 'SubmodelElementList' && !item.children" color="primary">
                        mdi-file-alert
                    </v-icon>
                    <!-- Icon for SubmodelelementList -->
                    <v-icon v-else-if="item.modelType === 'SubmodelElementList'" color="primary">mdi-list-box</v-icon>
                    <!-- Icon for empty Entities -->
                    <v-icon v-else-if="item.modelType === 'Entity' && !item.children" color="primary">
                        mdi-file-alert
                    </v-icon>
                    <!-- Icon for Entities -->
                    <v-icon v-else-if="item.modelType === 'Entity'" color="primary">mdi-format-list-group</v-icon>
                    <!-- Icon for every other SubmodelElement (like Property) -->
                    <v-icon v-else color="primary">mdi-file-code</v-icon>
                </template>
                <template #append="{ isActive }">
                    <v-chip
                        v-if="item.modelType"
                        color="primary"
                        size="x-small"
                        :style="{ 'margin-right': isActive ? '14px' : '' }"
                        >{{ item.modelType }}</v-chip
                    >
                    <!-- Button to Copy the Path to the Clipboard -->
                    <v-tooltip
                        v-if="isActive && (!editMode || item.modelType !== 'Submodel')"
                        text="Copy Path to Clipboard"
                        :open-delay="600"
                        location="bottom">
                        <template #activator="{ props }">
                            <v-icon
                                color="subtitleText"
                                v-bind="props"
                                @click.stop.prevent="copyPathToClipboard(item.path)"
                                >{{ copyIcon }}</v-icon
                            >
                        </template>
                    </v-tooltip>
                    <!-- Context menu for Submodels -->
                    <v-menu v-if="editMode && item.modelType === 'Submodel'">
                        <template #activator="{ props }">
                            <v-btn icon="mdi-dots-vertical" variant="plain" color="subtitleText" v-bind="props"></v-btn>
                        </template>
                        <v-sheet border>
                            <v-list dense density="compact" class="py-0">
                                <!-- Copy Path to Clipboard -->
                                <v-list-item @click="copyPathToClipboard(item.path)">
                                    <template #prepend>
                                        <v-icon size="x-small">{{ copyIcon }} </v-icon>
                                    </template>
                                    <v-list-item-subtitle>Copy Path</v-list-item-subtitle>
                                </v-list-item>
                                <!-- Open Submodel edit dialog -->
                                <v-list-item @click="openEditDialog()">
                                    <template #prepend>
                                        <v-icon size="x-small">mdi-pencil</v-icon>
                                    </template>
                                    <v-list-item-subtitle>Edit Submodel</v-list-item-subtitle>
                                </v-list-item>
                                <!-- Delete Submodel -->
                                <v-list-item @click="showDeleteDialog()">
                                    <template #prepend>
                                        <v-icon size="x-small">mdi-delete</v-icon>
                                    </template>
                                    <v-list-item-subtitle>Delete Submodel</v-list-item-subtitle>
                                </v-list-item>
                            </v-list>
                        </v-sheet>
                    </v-menu>
                </template>
            </v-list-item>
        </v-lazy>
        <!-- Recursive Treeview -->
        <template v-if="showChildren">
            <vTreeview
                v-for="innerItem in item.children"
                :key="innerItem.id"
                :item="innerItem"
                :depth="depth + 1"></vTreeview>
        </template>
    </div>
</template>

// TODO Transfer to composition API
<script lang="ts">
    import { defineComponent } from 'vue';
    import { useRoute, useRouter } from 'vue-router';
    import { useTheme } from 'vuetify';
    import { useReferableUtils } from '@/composables/AAS/ReferableUtils';
    import SubmodelElementHandling from '@/mixins/SubmodelElementHandling';
    import { useAASStore } from '@/store/AASDataStore';
    import { useNavigationStore } from '@/store/NavigationStore';
    import { extractEndpointHref } from '@/utils/DescriptorUtils';

    export default defineComponent({
        name: 'VTreeview',
        mixins: [SubmodelElementHandling],
        props: ['item', 'depth'],

        setup() {
            const theme = useTheme();
            const navigationStore = useNavigationStore();
            const aasStore = useAASStore();
            const route = useRoute();
            const router = useRouter();
            const { nameToDisplay } = useReferableUtils();

            return {
                theme, // Theme Object
                navigationStore, // NavigationStore Object
                aasStore, // AASStore Object
                route, // Route Object
                router, // Router Object
                nameToDisplay,
                extractEndpointHref,
            };
        },

        data() {
            return {
                showChildren: false,
                copyIcon: 'mdi-clipboard-file-outline',
            };
        },

        computed: {
            // Check if the current Device is a Mobile Device
            isMobile() {
                return this.navigationStore.getIsMobile;
            },

            // get selected AAS from Store
            SelectedAAS() {
                return this.aasStore.getSelectedAAS;
            },

            // Check if the current Route is the AAS Editor
            editMode() {
                return this.route.name === 'AASEditor';
            },
        },

        mounted() {
            // Check if the current items showChildren Property is true if it exists
            if ('showChildren' in this.item && this.item.showChildren) {
                this.showChildren = true;
            }
        },

        methods: {
            // Function to show/hide Children
            toggleChildren() {
                this.showChildren = !this.showChildren;
            },

            // Function to toggle a Node
            toggleNode() {
                // console.log('Selected Prop: ', this.item);
                // duplicate the selected Node Object
                let localItem = { ...this.item };
                // invert the isActive Property
                localItem.isActive = !localItem.isActive;
                // Add path of the selected Node to the URL as Router Query
                if (localItem.isActive) {
                    const aasEndpopint = this.extractEndpointHref(this.SelectedAAS, 'AAS-3.0');
                    if (this.isMobile) {
                        // Change to SubmodelElementView on Mobile and add the path to the URL
                        this.router.push({
                            name: 'SubmodelViewer',
                            query: {
                                aas: aasEndpopint,
                                path: localItem.path,
                            },
                        });
                    } else {
                        // just add the path to the URL
                        this.router.push({
                            query: {
                                aas: aasEndpopint,
                                path: localItem.path,
                            },
                        });
                    }
                    if (localItem.modelType !== 'Submodel') delete localItem.id;
                    this.aasStore.dispatchSelectedNode(localItem);
                } else {
                    // remove the path query from the Route entirely
                    let query = { ...this.route.query };
                    delete query.path;
                    this.router.push({ query: query });
                    this.aasStore.dispatchSelectedNode({});
                }
            },

            // Function to copy the path of the current Node to the Clipboard
            copyPathToClipboard(path: string) {
                // console.log('Copy Path to Clipboard: ', path);
                // set the icon to checkmark
                this.copyIcon = 'mdi-check';
                // copy the path to the clipboard
                navigator.clipboard.writeText(path);
                // set the clipboard tooltip to false after 1.5 seconds
                setTimeout(() => {
                    this.copyIcon = 'mdi-clipboard-file-outline';
                }, 2000);
                // open Snackbar to inform the user that the path was copied to the clipboard
                this.navigationStore.dispatchSnackbar({
                    status: true,
                    timeout: 2000,
                    color: 'success',
                    btnColor: 'buttonText',
                    text: 'Path copied to Clipboard.',
                });
            },

            openEditDialog() {
                this.$emit('openEditDialog', this.item);
            },

            showDeleteDialog() {
                this.$emit('showDeleteDialog', this.item);
            },
        },
    });
</script>

<style scoped>
    /* move the Treeview Text a few Pixels to the left for a better look */
    ::v-deep(.v-list-item__prepend) {
        margin-right: -16px !important;
    }
</style>

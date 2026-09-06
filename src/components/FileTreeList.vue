<template>
    <input type="file" webkitdirectory directory multiple @change="handleFolderUpload" />
    <div>
        <button @click="addRootFolder()">+'📁'</button>
        <Draggable v-model="data" :disableDrag="true" class="mtl-tree" ref="tree" virtualization style="height: 500px"
            treeLine>
            <template #default="{ node, stat }">
                <div id="wrapper">
                    <OpenIcon v-if="stat.data.class === 'folder'" :open="stat.open" class="mtl-mr"
                        @click.native="stat.open = !stat.open" />
                    <span v-else>&nbsp;&nbsp;&nbsp;&nbsp;</span>
                    <span>{{ (stat.data.class === 'folder' ? '📁' : '📄') }}</span>
                    <input class="mtl-checkbox mtl-mr" type="checkbox" v-model="stat.checked" />
                    <span v-if="!(stat.data.isEdit)" data-placeholder="enter name" class="mtl-ml"
                        @click="stat.open = !stat.open" @input="handleInput($event, node, stat)">{{
                            node.text }}</span>
                    <input v-else ref="inputRef" v-model="node.text" @keyup.enter="toggleEdit(stat)"
                        @blur="toggleEdit(stat)"></input>
                    <span>&nbsp;&nbsp;&nbsp;&nbsp;</span>
                    <button v-if="stat.data.class === 'folder'" @click="addFile(node, stat)">+'📃'</button>
                    <button v-if="stat.data.class === 'folder'" @click="addFolder(node, stat)">+'📁'</button>
                    <button @click="deleteNode(node, stat)">-</button>
                    <button @click="toggleEdit(stat)">
                        {{ stat.data.isEdit ? 'Save' : 'Edit' }}
                    </button>
                </div>
            </template>
        </Draggable>

    </div>
    <div class="actions">
        <button @click="addFile()">add: append to first node</button>
        <button @click="addAfterSecondNode()">add: after second node</button>
        <button @click="addNestedNewNodes()">add: nested new nodes</button>
        <button @click="addMulti()">addMulti</button>
        <br />
        <button @click="batchUpdate()">batchUpdate</button>
        <br />
        <button @click="getChecked(false)">getChecked(false)</button>
        <button @click="getChecked(true)">getChecked(true)</button>
        <br />
        <button @click="getDataAll()">getData: all</button>
        <button @click="getDataFirstNode()">getData: first node</button>
    </div>

</template>

<script lang="ts">

import '@he-tree/vue/style/default.css'
import '@he-tree/vue/style/material-design.css'
import { BaseTree, Draggable, pro, OpenIcon } from "@he-tree/vue";
import "@he-tree/vue/style/default.css";
import { reactive, defineComponent, onMounted } from "vue";
import data from "../data.json";
import fileTree from './FileTree.vue';
import { ref } from "vue";


// Define the prop to accept the data


const isEditing = ref(false)

export default defineComponent({
    components: { Draggable, OpenIcon },
    data() {
        return {
            data,
            isEditing,
        };
    },
    methods: {
        addRootFolder(this: any) {
            this.data.push({
                "text": "newFolder",
                "class": 'folder',
                "children": [],
                "isEdit": false,
            })
            this.$refs.tree.add(
                { text: 'newFolder', class: 'folder' },
            )
        },
        buildTree(files: FileList | null): any[] {
            const root: any[] = []
            if (!files || !files.length) {
                return [];
            }
            for (const file of files) {

                const parts = file.webkitRelativePath.split('/')

                let currentLevel = root

                for (let i = 0; i < parts.length; i++) {
                    const part = parts[i]
                    let existingNode = currentLevel.find(node => node.text === part)

                    if (!existingNode) {
                        existingNode = {
                            "text": part,
                            "class": i < parts.length - 1 ? 'folder' : 'file',
                            "children": i < parts.length - 1 ? [] : null,
                            "isEdit": false,
                        }
                        currentLevel.push(existingNode)
                    }
                    if (existingNode.children) {
                        currentLevel = existingNode.children
                    }
                }
            }

            return root
        },
        handleFolderUpload(event: Event) {
            const files: FileList | null = (event.target as HTMLInputElement).files
            this.data = this.buildTree(files)

        },
        toggleEdit(stat: any) {
            stat.data.isEdit = !stat.data.isEdit
        },
        notify() {
            alert(`Outputed to browser console, please check`)
        },
        deleteNode(this: any, node?: any, stat?: any) {
            this.$refs.tree.remove(stat)
        },
        handleInput(event: InputEvent, node: any, stat: any) {
            let input: string = event.data ? event.data : '';
            node.text = (event.target as HTMLInputElement).innerText;

        },
        addFolder(this: any, node?: any, stat?: any) {
            this.$refs.tree.add(
                { text: 'new node', class: 'folder' },
                stat,
                0
            )
        },
        addFile(this: any, node?: any, stat?: any) {
            this.$refs.tree.add(
                { text: 'new node', class: 'file' },
                stat,
                0
            )
        },
        addAfterSecondNode(this: any) {
            this.$refs.tree.add({ text: 'new node' }, null, 2)
        },
        addNestedNewNodes(this: any) {
            this.$refs.tree.add(
                { text: 'new parent', children: [{ text: 'new child' }] },
                null,
                2
            )
        },
        addMulti(this: any) {
            // nested new nodes supported
            this.$refs.tree.addMulti(
                [{ text: 'addMulti1' }, { text: 'addMulti2' }],
                this.$refs.tree.rootChildren[1],
                0
            )
        },
        batchUpdate(this: any) {
            this.$refs.tree.batchUpdate(() => {
                this.addAppendToFirstNode()
                this.addMulti()
            })
        },
        getChecked(this: any, withDemi: boolean) {
            this.notify()
        },
        getDataAll(this: any) {
            this.notify()
        },
        getDataFirstNode(this: any) {
            this.notify()
        },
    },

    mounted() { },
});
</script>
<style>
span {
    display: inline-block;
    min-width: 50px;
    white-space: nowrap;
}

span:empty::before {
    content: attr(data-placeholder);
    color: #888888;
    /* Light gray placeholder color */
    cursor: text;
}

#wrapper {
    border: 1px solid #ccccccc3;
    padding: 5px;
}

button {
    margin-left: 15px;
}
</style>
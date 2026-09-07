<script setup lang="ts">


import { ref } from 'vue'

interface tree_node {
    text: string,
    class: string,
    isEdit: boolean,
    children: tree_node[],
}

const fileTree = ref<any[]>([]);
defineExpose({
    fileTree,
})

// Convert flat file list to a nested tree
function buildTree(files: FileList | null): any[] {
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
                    "draggable": false,
                    "droppable": false,
                }
                currentLevel.push(existingNode)
            }
            if (existingNode.children) {
                currentLevel = existingNode.children
            }
        }
    }

    return root
}

function handleJson(data: any) {

    let curNode = data[0];
    let queue = [curNode];
    while (queue.length > 0) {
        let node = queue.shift();
        if (node) {
            node['text'] = node['name'];
            node['class'] = node.children ? 'folder' : 'file';
            node['isEdit'] = false;
            for (let child of (node.children || [])) {
                queue.push(child);
            }
            delete node.name;
            delete node.path;
            delete node.file;
        }
    }
    return data;
}

function handleFolderUpload(event: Event) {
    const files: FileList | null = (event.target as HTMLInputElement).files
    fileTree.value = buildTree(files)

    //let data = handleJson(fileTree.value)

}
</script>

<template>
    <div class="upload-container">
        <input type="file" webkitdirectory directory multiple @change="handleFolderUpload" />

        <div v-if="fileTree.length" class="tree-view">
            <h3>File Tree:</h3>
            <ul>
                <!-- Recursive template or sub-component for deep trees -->
                <li v-for="node in fileTree">
                    <span>{{ node.children ? '📁' : '📄' }} {{ node.text }}</span>
                    <ul v-if="node.children && node.children.length">
                        <li v-for="child in node.children">
                            {{ child.children ? '📁' : '📄' }} {{ child.text }}
                            <ul v-if="child.children && child.children.length">
                                <li v-for="child_child in child.children">
                                    {{ child_child.children ? '📁' : '📄' }} {{ child_child.text }}
                                    <ul v-if="child_child.children && child_child.children.length">
                                        <li v-for="child_child_ in child_child.children">
                                            {{ child_child_.children ? '📁' : '📄' }} {{ child_child_.text }}

                                        </li>
                                    </ul>
                                </li>
                            </ul>
                        </li>
                    </ul>
                </li>
            </ul>
        </div>
    </div>
</template>

<script lang="ts">

</script>
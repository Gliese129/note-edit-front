<template>
	<div class="home">
		<h3>文件一览:</h3>
		<el-input v-model="filterText" placeholder="请输入需要查找的文件" />
		<el-tree
			ref="catlog"
			:props="treeProps"
			:load="loadNode"
			lazy
			@node-click="handldNodeClicked"
			:filter-node-method="filterNode"
		>
			<template #default="{ node, data }">
				<file-view :name="data.name" :type="data.type" :opened="node.expanded" />
			</template>
		</el-tree>
	</div>
</template>

<script lang="ts" setup>
import FileView from '@/components/FileView.vue'
import { reactive, watch } from 'vue'
import type { ITreeNode } from '@/global/entity/Tree'
import fileApis from '@/apis/fileApis'
import { fileInfoToNode } from '@/utils/file'
import { useRouter } from 'vue-router'
import { useStore } from 'vuex'
const router = useRouter()
const store = useStore()

let filterText = $ref('')
let catlog = $ref<any>(null)
watch($$(filterText), (value: string) => {
	if (catlog) {
		catlog.filter(value)
	}
})
const treeProps = reactive({
	label: 'name',
	children: 'children',
	isLeaf: 'leaf'
})

const filterNode = (filter: string, node: ITreeNode) => {
	return node.name.includes(filter)
}
const loadNode = (node: any, resolve: any) => {
	let nodeData = node.data
	if (node.level === 0) {
		// * 加载根目录
		fileApis.fetchRootFolders().then((response) => {
			let data = response.obj
			resolve(fileInfoToNode(data))
		})
	} else {
		fileApis.fetchChildrenFiles(nodeData.id).then((response) => {
			let data = response.obj
			resolve(fileInfoToNode(data))
		})
	}
}
const handldNodeClicked = (node: ITreeNode) => {
	// * check if node is file
	if (node.type !== 'folder') {
		store.commit('file/setFileInfoLight', {
			id: node.id,
			name: node.name
		})
		router.push(`/note/view/${node.id}`)
	}
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss"></style>

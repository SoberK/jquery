<template>
	<div :class="{'upload':!$slots.default}">
		<el-upload
			ref='elUpload'
			action=""
			multiple
			:data="param"
			:http-request="uploadFn"
			:on-success="uploadSuccess"
			:on-error="uploadError"
			:show-file-list="false"
			accept=".doc,.docx,.xls,.xlsx,.pdf,.jpg,.png,.rar.zip"
			name="multipartFile"
		>
			<slot></slot>
			<template slot="trigger" v-if='!$slots.default'>
-        <el-button class="mm-btn-blue-submit">
+        <el-button  class="mm-btn-blue-submit">
					<svg-icon
						icon-class="xingzhuang"
						style="margin-right: 5px"
					/>
					{{text}}
				</el-button>
-        <span class="filelimit"> <span style="color:#FB6060"> * </span><span class="text">支持扩展名：.doc、.docx、.xls、.xlsx、.pdf、.jpg、.png、.rar、.zip，文件大小不超过200M</span></span>
+        <div style='margin-top: 10px' class="filelimit"> <span class="text">支持扩展名：.doc、.docx、.xls、.xlsx、.pdf、.jpg、.png、.rar、.zip，文件大小不超过200M</span></div>
			</template>

			<!--        <span style="margin-left: 10px">*支持扩展名：.doc、.docx、.pdf、.png、.bmp,文件大小不能超过10M</span>-->
		</el-upload>
		<div>
			<template v-for="(item, index) in fileList">
				<el-row
					class="filelist"
					:key="index"
					v-if="index % 3 === 0"
					:gutter="24"
				>
					<template v-for="(subItem, subIndex) in fileList">
						<el-col
							:key="subIndex"
							v-if="subIndex >= index && subIndex < index+3 "
							:span="7"
							class="mm-ly--flex mm-ly--between mm-m-bottom--15"
						>
							<div class="file-percentage" v-if="subItem.slicing && subItem.static === 'load'">
								<div class="file-speed" :style="{width:subItem.percentage+'%'}"></div>
							</div>
							<div class="file-title mm-ly--flex">
								<!-- wendang -->
								<svg-icon
									:icon-class="fileTypeCheck(subItem).icon"
									style="margin-right: 5px;font-size:18px"
									class="mm-ly--flex-shrink"
								/>
								<div class="file-title">{{subItem.fileName}}</div>
							</div>
							<i
								@click.stop="removeFile(subIndex)"
								class="el-icon-close icon-close"
							></i>
							<!-- 图标加载中 -->
							<i
								class="el-icon-loading icon-loading"
								v-if="subItem.static === 'load' && !subItem.slicing"
							></i>
							<!-- 文字 百分比加载中 -->
							<span
								class="text-loding"
								style="color: #666666;font-size: 10px;"
								v-if="subItem.static === 'load' && subItem.slicing "> {{ (Math.floor(subItem.percentage) || 0) + '%'}} </span>

							<i
								class="el-icon-warning icon-warning"
								v-else-if="subItem.static === 'error'"
							></i>
							<i
								class="el-icon-success icon-success"
								v-else-if="subItem.static === 'success'"
							></i>
						</el-col>
					</template>
				</el-row>
			</template>
		</div>
	</div>
</template>

<script>
import { uploadComplete } from '@/api/common'
import { fileTypeCheck } from '@/utils/typeCheck'
import axios from 'axios'

export default {
	name: 'AttachUpload',
	props: {
		// 文件分片大小 默认10MB分一片
		bytesPerPiece: {
			type: Number,
			default: () => 10
		},
		// 线程 默认4条线程
		thread: {
			type: Number,
			default: () => 4
		},
		value: {
			type: Object,
			default: () => { }
		},
		text: {
			type: String,
			default: '上传文件'
		}
	},
	data () {
		return {
			fileSliceSize: 0, // 文件分片大小 默认10MB分一片
			param: {
				appId: 'smartCourt',
				bucketName: 'execution-record',
				spaceSize: null,
				fileName: null,
				contentType: null
			},
			time: 0,

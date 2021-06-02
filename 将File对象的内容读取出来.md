需求：通过使用**elementUI**的`el-upload`组件上传文件，得到的数据为`File对象`，但是只需要文件的内容

```html
<el-upload
      class="upload-demo"
      action="/api/note/add"
      :on-change="handleChange"
      :file-list="fileList"
      :http-request="uploadFile"
    >
      <el-button size="small" type="primary">点击上传</el-button>
      <template #tip>
        <div class="el-upload__tip">只能上传 .md 结尾的文件</div>
      </template>
    </el-upload>
```

```js
const uploadFile = (param) => {
      console.log(param.file)  // File {uid: 1621508157622, name: "AD Involved.txt", lastModified: 1575795169060, lastModifiedDate: Sun Dec 08 2019 16:52:49 GMT+0800 (中国标准时间), webkitRelativePath: "", …}
      console.log('on upload')
    }
```

解决方法：使用`FileReader`将`File`对象的内容以字符串读取出来

```js
const uploadFile = (param) => {
      const formData = new FormData()
      console.log(param.file)
      let reader = new FileReader()
      let baseData
      reader.readAsText(param.file, 'utf8')
      reader.onload = () => {
        baseData = reader.result
        console.log(baseData)  // 输出文件的内容
      }
      console.log('on upload')
    }
```


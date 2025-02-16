<template>
    <div>
        <div class="custom-file-input">
            <input type="file" @change="handlefile" accept=".docx" id="fileInput" />
            <label for="fileInput">选择文件</label>
        </div>
        <div class="down">
            <button class="download-btn" @click="downloadFile">下载文件</button>
        </div>
        <div class="content">
            <span>预览</span>
            <div v-html="forhtml" class="show"></div>
        </div>
    </div>
</template>

<script>
import mammoth from "mammoth";
import { saveAs } from "file-saver";

export default {
    data() {
        return {
            forhtml: "",
        };
    },
    methods: {
        async handlefile(event) {
            const file = event.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = async (e) => {
                const arrayBuffer = e.target.result;

                const options = {
                    convertImage: mammoth.images.imgElement(async (image) => {
                        console.log("[DEBUG] 开始处理图片，类型:", image.contentType);
                        try {
                            const base64Data = await image.read("base64");
                            const src = `data:${image.contentType};base64,${base64Data}`;
                            return { src };
                        } catch (err) {
                            console.error("[ERROR] 图片处理失败:", err);
                            return { src: "" };
                        }
                    }),
                };

                try {
                    const result = await mammoth.convertToHtml(
                        { arrayBuffer: arrayBuffer },
                        options
                    );
                    this.forhtml = result.value;
                    console.log("[DEBUG] 生成的 HTML:", this.forhtml);
                } catch (err) {
                    console.error("[ERROR] DOCX 转换失败:", err);
                }
            };
            reader.readAsArrayBuffer(file);
        },
        downloadFile() {
            const randomNum = Math.floor(Math.random() * 10000);
            const timestamp = Date.now();
            const fileName = `${randomNum}-${timestamp}.html`;
            const blob = new Blob([this.forhtml], { type: "text/html;charset=utf-8" });
            saveAs(blob, fileName);
        },
    },
};
</script>

<style lang="css" scoped>
.custom-file-input {
    position: relative;
    display: inline-block;
    margin: 0 auto;
    width: 1000px;
    height: 100px;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.custom-file-input input[type="file"] {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    opacity: 0;
    cursor: pointer;
}

.custom-file-input label {
    display: inline-block;
    background-color: #007BFF;
    color: white;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
}

.custom-file-input label:hover {
    background-color: #0056b3;
}

.content {
    margin: 0 auto;
    margin-top: 50px;
    width: 1000px;
    border: 1px solid #404040;
    border-radius: 10px;
    display: flex;
    flex-direction: column;
    align-items: center;
    flex-wrap: wrap;
    position: relative;
}

span {
    color: blue;
    font-size: 24px;
    font-weight: bold;
}

.show {
    width: 1000px;
}

.download-btn {
    background-color: #007BFF;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
    margin-top: 20px;
}

.download-btn:hover {
    background-color: #0056b3;
}

.down {
    width: 1000px;
    margin: 0 auto;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
}
</style>
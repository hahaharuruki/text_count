<template>
    <div class="text-counter-container">
        <div class="text-counter" @dragover.prevent @drop.prevent="handleDrop">
            <div class="advertisement">
                <img src="ad_image.png" alt="広告スペース" class="ad-image">
                <ins class="adsbygoogle"
                    style="display:block"
                    data-ad-client="ca-pub-XXXXXX"
                    data-ad-slot="XXXXXX"
                    data-ad-format="auto"></ins>
            </div>
            <h1 class="title">文字数カウント</h1>
            <p class="description">
                テキストボックスに入力された文字数をカウントします。テキストファイル、Wordファイル内の文字列もカウントできます。
            </p>
            <div class="buttons-top">
                <button @click="pasteText" class="paste-button">ペースト</button>
                <button @click="clearText" class="clear-button">リセット</button>
            </div>
            <textarea v-model="text" @input="updateCounts" class="text-area"
                placeholder="カウントしたい文字列を入力、貼り付け、またはファイルのドラッグ&ドロップをしてください。"></textarea>
            <div class="options">
                <div class="counts-left">
                    <div class="count-item">
                        <p class="count-label">文字数</p>
                        <p class="count-result">{{ totalChars }}</p>
                    </div>
                    <div class="count-item">
                        <p class="count-label">改行を除いた文字数</p>
                        <p class="count-result">{{ charsWithoutNewlines }}</p>
                    </div>
                    <div class="count-item">
                        <p class="count-label">空白、改行を除いた文字数</p>
                        <p class="count-result">{{ charsWithoutSpacesAndNewlines }}</p>
                    </div>
                </div>
            </div>
            <input type="file" @change="handleFileUpload" class="file-upload" accept=".txt,.docx" />
            <div v-if="fileName">アップロードされたファイル: {{ fileName }}</div>
            <div class="advertisement">
                <img src="ad_image.png" alt="広告スペース" class="ad-image">
                <ins class="adsbygoogle"
                    style="display:block"
                    data-ad-client="ca-pub-XXXXXX"
                    data-ad-slot="XXXXXX"
                    data-ad-format="auto"></ins>
            </div>
        </div>
    </div>
</template>

<script>
import mammoth from "mammoth";

export default {
    data() {
        return {
            text: '',
            totalChars: 0,
            charsWithoutSpaces: 0,
            charsWithoutNewlines: 0,
            charsWithoutSpacesAndNewlines: 0,
            fileName: ''
        };
    },
    watch: {
        text() {
            this.updateCounts();
        }
    },
    methods: {
        async pasteText() {
            try {
                const clipText = await navigator.clipboard.readText();
                if (!clipText) {
                    console.error('クリップボードにテキストがありません。');
                    return;
                }
                this.text = clipText;
                this.updateCounts();
            } catch (error) {
                console.error("クリップボードの内容を読み取れませんでした: ", error);
            }
        },
        clearText() {
            this.text = '';
            this.updateCounts();
        },
        updateCounts() {
            this.totalChars = this.text.length;
            this.charsWithoutNewlines = this.text.replace(/\n/g, '').length; // 改行を除くが空白は除かない
            this.charsWithoutSpacesAndNewlines = this.text.replace(/[\s\n]/g, '').length; // 空白と改行を除く
        },
        handleFileUpload(event) {
            const file = event.target.files[0];
            this.fileName = file.name;
            this.readFile(file);
        },
        handleDrop(event) {
            const file = event.dataTransfer.files[0];
            this.fileName = file.name;
            this.readFile(file);
        },
        async readFile(file) {
            if (file.name.endsWith('.docx')) {
                this.readDocxFile(file);
            } else if (file.name.endsWith('.txt')) {
                this.readTextFile(file);
            } else {
                alert("対応していないファイル形式です。txt、docxファイルのみ対応しています。");
            }
        },
        async readDocxFile(file) {
            const reader = new FileReader();
            reader.onload = async (e) => {
                const arrayBuffer = e.target.result;
                try {
                    const result = await mammoth.extractRawText({ arrayBuffer });
                    this.text = result.value;
                    this.updateCounts();
                } catch (error) {
                    console.error("Wordファイルの読み取りエラー:", error);
                }
            };
            reader.readAsArrayBuffer(file);
        },
        readTextFile(file) {
            const reader = new FileReader();
            reader.onload = (e) => {
                this.text = e.target.result;
                this.updateCounts();
            };
            reader.readAsText(file);
        }
    }
};
</script>

<style scoped>
.text-counter-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  height: 100vh; /* ビューポートの高さに合わせる */
  overflow-y: auto; /* 縦方向のスクロールを有効にする */
}

.text-counter {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  flex-grow: 1;
  width: 100%;
}

.title {
  font-size: 24px;
  margin-bottom: 10px;
  font-weight: bold; /* タイトルを太字に */
}

.description {
  font-size: 16px;
  margin-bottom: 20px;
  text-align: left; /* 左詰めに */
}

.buttons-top {
  display: flex;
  justify-content: center;
  margin-bottom: 10px;
  gap: 10px;
}

.text-area {
  width: 100%;
  height: 225px; /* 高さを固定 */
  padding: 15px; /* 元の1.5倍 */
  border: 1px solid #ccc;
  border-radius: 5px;
  margin-bottom: 10px;
  font-size: 1em; /* デフォルトの文字サイズに戻す */
  box-sizing: border-box;
  resize: vertical; /* ユーザーが高さを調整可能にする */
  flex-grow: 1; /* 縦方向のスペースを適切に分配 */
}

.text-area::placeholder {
  font-size: 1em; /* プレースホルダーの文字サイズをデフォルトに戻す */
}

.options {
  display: flex;
  justify-content: space-between;
  width: 100%;
  margin-bottom: 20px;
}

.counts-left {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.count-item {
  display: flex;
  justify-content: space-between;
  width: 280px; /* 幅を280pxに統一 */
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.count-label {
  font-weight: bold; /* ラベルを太字に */
}

.count-result {
  font-weight: bold; /* カウント結果を太字に */
}

.buttons {
  display: flex;
  flex-direction: column;
  align-items: center; /* 中央に配置 */
  gap: 10px;
  margin-left: auto;
}

.paste-button, .clear-button {
  padding: 8px 16px; /* 元のサイズに戻す */
  font-size: 14px; /* 元のサイズに戻す */
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.paste-button {
  background-color: #3490dc;
  color: white;
}

.clear-button {
  background-color: #e3342f;
  color: white;
}

.advertisement {
  width: 100%;
  height: 100px; /* 高さを固定 */
  margin-bottom: 20px;
}

.ad-image {
  width: 100%;
  height: 100%; /* 高さを固定 */
  object-fit: cover;
}

.counts {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.file-upload {
  margin-top: 10px;
}

/* メディアクエリの追加 */
@media (max-width: 768px) {
  .buttons-top {
    flex-direction: row; /* 横並び */
    justify-content: center;
    width: 100%;
    gap: 10px;
  }

  .options {
    flex-direction: column;
    align-items: center;
  }

  .counts-left {
    width: 100%;
  }

  .count-item {
    width: 100%;
  }

  .buttons {
    width: 100%;
    flex-direction: row;
    justify-content: space-around;
    margin-left: 0;
  }
}

@media (max-width: 480px) {
  .title {
    font-size: 20px;
  }

  .description {
    font-size: 14px;
  }

  .text-area {
    height: 150px;
    padding: 10px;
    font-size: 0.9em;
  }

  .paste-button, .clear-button {
    padding: 6px 12px;
    font-size: 12px;
  }

  .buttons-top {
    flex-direction: row; /* 横並び */
    justify-content: center;
    width: 100%;
    gap: 10px;
  }
}

/* 広告の表示をスマホの画面でも確認できるように */
@media (max-width: 768px) {
  .advertisement {
    display: block;
  }
}
</style>
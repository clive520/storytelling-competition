# Storytelling Competition Webpage

說故事比賽作品集網頁 - 支援逐句語音播放與文字高亮同步

## 功能特色

- **逐句播放**: 點選任一句子即可從該句開始播放
- **文字高亮同步**: 播放時自動高亮當前句子，已播放句子變灰
- **性別化語音**: 女生使用女聲，男生使用男聲 (透過 Web Speech API)
- **語速可調**: 0.5x - 1.5x，預設 0.7x (適合練習聽力)
- **進度條顯示**: 即時顯示播放進度
- **免費無需 API Key**: 使用瀏覽器內建 Web Speech API
- **響應式設計**: 手機、平板、桌機皆可使用

## 學生名單

| 學號 | 姓名 | 性別 | 作品標題 |
|------|------|------|----------|
| 60124 | 何禹潼 | 女 | The Weight of Air |
| 60120 | 蔡亞璇 | 女 | The Boy and the Apple Tree |
| 60122 | 陳柏舜 | 男 | Leo and the Time Radio |
| 60115 | 林云晴 | 女 | The Magic Seed |
| 60126 | 吳丹楓 | 女 | The Bowl of Soup |

## 部署到 GitHub Pages

1. 建立 GitHub Repository
2. 將 `index.html` 推送到 `main` 分支
3. 在 Repository Settings > Pages 中啟用 GitHub Pages
4. 選擇 Deploy from branch: `main` / `/ (root)`
5. 幾分鐘後即可在 `https://<username>.github.io/<repo-name>/` 存取

## 本地測試

直接用瀏覽器開啟 `index.html` 即可 (建議使用 Chrome 或 Edge 獲得最佳語音品質)

## 語音引擎說明

- 使用瀏覽器內建 `SpeechSynthesis` API
- 不需網路連線 (語音引擎在本地)
- 語音品質取決於作業系統/瀏覽器內建語音包
- 建議使用 Chrome/Edge 獲得最佳中英文混合語音支援
- Windows: Microsoft David (男) / Microsoft Zira (女)
- macOS: Alex (男) / Samantha (女)

## 自訂學生資料

編輯 `index.html` 中的 `students` 陣列：

```javascript
const students = [
  {
    id: '學號',
    name: '姓名',
    gender: 'male' | 'female',
    title: '作品標題',
    story: '故事內容...'
  },
  // ...
];
```
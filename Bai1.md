# 📦 Bài 7: AssetBundle trong Unity

## 1. AssetBundle là gì?

**AssetBundle** là một tính năng trong Unity cho phép bạn đóng gói và phân phối các tài sản (assets) như hình ảnh, âm thanh, prefab, model 3D,... thành các tệp `.assetbundle`.  
Nó hỗ trợ tải tài sản theo yêu cầu, giúp tối ưu hiệu suất và giảm kích thước ứng dụng.

---

## 2. Tại sao nên sử dụng AssetBundle?

### 🎯 Tối ưu hiệu suất
- Chỉ tải khi cần → giảm thời gian khởi động
- Tiết kiệm RAM

### 🗂 Quản lý tài sản hiệu quả
- Nhóm asset cùng loại
- Phân chia dễ bảo trì

### 🔁 Cập nhật linh hoạt
- Chỉ cần cập nhật bundle → không phải rebuild toàn bộ app

### 🌐 Hỗ trợ tải từ xa
- Có thể lưu trên server và tải về khi cần

### 🔃 Dùng lại giữa nhiều dự án
- Tạo thư viện asset dùng chung

### 🧩 Quản lý phiên bản
- Dễ version hóa bằng cách thêm hash vào tên

---

## 3. Cài đặt AssetBundle Browser

Dùng để quản lý và build các asset bundle:

📦 **Link tải:**  
https://github.com/Unity-Technologies/AssetBundles-Browser.git

### Các bước:
1. Clone hoặc import qua Unity Package Manager
2. Mở `Window → AssetBundle Browser`

---

## 4. Cách đóng gói AssetBundle

### Bước 1: Gán tên bundle
- Trong `Inspector` của asset → nhập vào trường `AssetBundle` tên và Group

### Bước 2: Build bundle
- Mở AssetBundle Browser
- Chuyển tab `Build`
- Chọn tùy chọn phù hợp và bấm **Build**

<img width="1919" height="1006" alt="Ảnh chụp màn hình 2025-07-10 190353" src="https://github.com/user-attachments/assets/111ac45d-709e-4884-9a56-cb739ab05cf8" />

---

## 5. Các tùy chọn khi Build

| Tùy chọn | Mô tả |
|---------|------|
| `Standard Compression (LZMA)` | Nén tốt, load chậm hơn |
| `LZ4` | Nén nhẹ, load nhanh |
| `Uncompressed` | Không nén, file lớn |
| `Exclude Type Information` | Không chứa thông tin kiểu, nhẹ hơn |
| `Force Rebuild` | Bắt buộc build lại toàn bộ |
| `Ignore Type Tree Changes` | Bỏ qua thay đổi kiểu asset |
| `Append Hash` | Gắn hash vào tên file để quản lý version |
| `Strict Mode` | Kiểm tra chặt chẽ, cảnh báo lỗi nếu có |
| `Dry Run Build` | Chỉ kiểm tra cấu hình, không build thật |

---

## 6. Vị trí lưu AssetBundle

Kết quả build nên lưu trong thư mục:
Assets/StreamingAssets/ khi xài copy StreamingAssets

## 7.Lưu ý 
- Các thư mục có tệp con phụ thuộc kèm theo phải kèm cả các phụ thuộc đó vào .bundle
- thư mục đầu tiên trong 4 cái mới là name của .bundle
  
<img width="470" height="185" alt="Ảnh chụp màn hình 2025-07-10 190556" src="https://github.com/user-attachments/assets/1fd5ee8a-0011-4e84-8706-6b9aa20499bd" />

## 8. Code load Asset từ AssetBundle mẫu 
```csharp
using UnityEngine;
using System.IO;

public class Assetbundle : MonoBehaviour
{
    [Header("Tên file bundle (không có đuôi .bundle)")]
    public string assetBundleName; // VD: "character"
    
    [Header("Tên prefab trong bundle")]
    public string prefabName;      // VD: "Monster_1_Salamander"

    private AssetBundle assetBundle;
    private GameObject instance;

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.L))
        {
            LoadAndInstantiatePrefab();
        }

        if (Input.GetKeyDown(KeyCode.R))
        {
            if (instance != null)
            {
                Destroy(instance);
                Debug.Log("♻️ Đã huỷ đối tượng hiện tại");
            }
        }
    }

    private void LoadAndInstantiatePrefab()
    {
        // Nếu bundle đã load → Unload lại
        if (assetBundle != null)
        {
            Debug.LogWarning("AssetBundle đã được load trước đó. Đang Unload...");
            assetBundle.Unload(false);
            assetBundle = null;
        }

        string bundlePath = Path.Combine(Application.streamingAssetsPath, assetBundleName);
        Debug.Log($"📦 Load AssetBundle từ: {bundlePath}");

        // Load từ file
        assetBundle = AssetBundle.LoadFromFile(bundlePath);

        if (assetBundle == null)
        {
            Debug.LogError($"❌ Không thể load AssetBundle: {assetBundleName}");
            return;
        }

        Debug.Log($"✅ AssetBundle '{assetBundleName}' đã được load");

        // Load prefab
        GameObject prefab = assetBundle.LoadAsset<GameObject>(prefabName);

        if (prefab == null)
        {
            Debug.LogError($"❌ Không tìm thấy Prefab '{prefabName}' trong bundle '{assetBundleName}'");
            return;
        }

        // Instantiate
        instance = Instantiate(prefab);
        instance.name = prefab.name;

        Debug.Log($"✅ Đã Instantiate Prefab: {prefab.name}");
    }

    private void OnDestroy()
    {
        if (assetBundle != null)
        {
            assetBundle.Unload(false);
            assetBundle = null;
        }

        if (instance != null)
        {
            Destroy(instance);
        }
    }
}


```
<img width="1919" height="980" alt="Ảnh chụp màn hình 2025-07-10 190416" src="https://github.com/user-attachments/assets/e770c9c8-beb6-4f05-ba85-3dcfe2037040" />
## 9. 🛠️ Các hàm và phương thức thường dùng trong AssetBundle

Dưới đây là các API phổ biến nhất trong quá trình sử dụng AssetBundle trong Unity (C#), chia thành nhóm theo mục đích sử dụng:

---

### 🎯 Nhóm tải AssetBundle

| Hàm / Phương thức | Mô tả |
|-------------------|------|
| `AssetBundle.LoadFromFile(path)` | Tải trực tiếp từ file `.assetbundle` ở local (ví dụ: từ `StreamingAssets`) |
| `AssetBundle.LoadFromMemory(byte[])` | Tải từ mảng byte (thường dùng khi tải qua mạng rồi parse bằng `UnityWebRequest`) |
| `AssetBundle.LoadFromStream(Stream)` | Tải từ stream, dùng cho advanced I/O |

---

### 📦 Nhóm load asset từ AssetBundle

| Hàm | Mô tả |
|-----|------|
| `LoadAsset<T>("assetName")` | Load asset có kiểu cụ thể từ bundle (thường là `GameObject`, `TextAsset`, `AudioClip`,...) |
| `LoadAllAssets<T>()` | Load tất cả asset có kiểu `T` |
| `LoadAllAssets()` | Load toàn bộ asset, không phân biệt kiểu |
| `LoadAssetAsync<T>("assetName")` | Load asset bất đồng bộ (async), thích hợp khi dùng coroutine |
| `LoadAllAssetsAsync<T>()` | Load tất cả asset kiểu `T` async |

---

### 🧼 Nhóm giải phóng bộ nhớ

| Hàm | Mô tả |
|-----|------|
| `assetBundle.Unload(bool unloadAllLoadedObjects)` | Giải phóng AssetBundle khỏi bộ nhớ. Nếu `true` thì giải phóng luôn asset đã load |

---

### 🌐 Nhóm tải AssetBundle từ mạng

> Kết hợp `UnityWebRequestAssetBundle` từ `UnityEngine.Networking`

```csharp
UnityWebRequest uwr = UnityWebRequestAssetBundle.GetAssetBundle(url);
yield return uwr.SendWebRequest();

AssetBundle bundle = DownloadHandlerAssetBundle.GetContent(uwr);
```
### 🧪 Nhóm kiểm tra & tiện ích AssetBundle

| **Phương thức**            | **Mô tả**                                                         |
|----------------------------|--------------------------------------------------------------------|
| `Contains("assetName")`    | Kiểm tra xem AssetBundle có chứa asset tên `"assetName"` không    |
| `GetAllAssetNames()`       | Trả về mảng chứa toàn bộ tên asset trong AssetBundle              |
| `GetAllScenePaths()`       | Trả về mảng chứa toàn bộ đường dẫn scene trong AssetBundle        |

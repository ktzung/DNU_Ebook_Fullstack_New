# Hướng Dẫn: Xây Dựng Ứng Dụng Vue 3 CRUD với MockAPI.io (Online)

Ở bài trước (bài 16), chúng ta đã dùng `json-server` để chạy một API giả trên máy tính cá nhân (localhost). Cách đó rất tốt để code offline.
Tuy nhiên, trong thực tế hoặc khi muốn chia sẻ ứng dụng cho người khác dùng thử, ta cần một API "trên mây" (Cloud). **MockAPI.io** là một công cụ miễn phí tuyệt vời để làm việc này.

Bài hướng dẫn này sẽ giúp bạn **thay thế json-server bằng MockAPI.io**. Các phần code về giao diện (Vue) gần như giữ nguyên, chứng tỏ sức mạnh của việc tách biệt API Service.

---

## Phần 1: Thiết Lập MockAPI.io

Thay vì tạo file `db.json`, ta sẽ tạo database trên web.

### Bước 1: Đăng ký tài khoản
1.  Truy cập [https://mockapi.io/](https://mockapi.io/).
2.  Đăng nhập bằng tài khoản Google hoặc GitHub.

### Bước 2: Tạo Project
1.  Tại màn hình Dashboard, bấm nút **"+"** (hoặc **"New Project"**).
2.  Nhập tên Project: `VueCrudApp`.
3.  API Prefix: Để mặc định (thường là `/api/v1`).
4.  Bấm **Create**.

### Bước 3: Tạo Resource (Tạo bảng dữ liệu)
Project vừa tạo đang trống, ta cần tạo bảng `products`.
1.  Bấm vào project **VueCrudApp** vừa tạo.
2.  Bấm nút **"New Resource"**.
3.  Điền thông tin:
    - **Resource Name**: `products` (Lưu ý viết thường, số nhiều).
4.  Định nghĩa Schema (Các trường dữ liệu):
    - Mặc định đã có `id`, `createdAt`.
    - Thêm các trường sau:
        - `name` (Type: startCase / String)
        - `price` (Type: Number / Money)
        - `image` (Type: Faker.js -> Image -> avatar/image)
        - `description` (Type: String)
5.  Bấm **Create**.

### Bước 4: Lấy Endpoint (Đường dẫn API)
Sau khi tạo xong, bạn sẽ thấy một đường link dạng:
`https://67...mockapi.io/api/v1/products`
-> Hãy copy phần gốc: `https://67...mockapi.io/api/v1` ( Bỏ chữ `/products` đuôi đi).
*Ví dụ ID của bạn là `677f805d0476123f76a6b5a1`, thì URL là `https://677f805d0476123f76a6b5a1.mockapi.io/api/v1`.*

---

## Phần 2: Khởi Tạo Dự Án Vue

*(Bước này giống hệt bài 16, nhưng không cần cài json-server)*

### Bước 1: Tạo Project
```bash
npm init vue@latest vue-mockapi-app
# Chọn: No TypeScript, Yes Router, Yes ESLint/Prettier
```

### Bước 2: Cài Axios
```bash
cd vue-mockapi-app
npm install
npm install axios
# KHÔNG CẦN cài json-server nữa!
```

---

## Phần 3: Cấu Hình API Service (Quan Trọng)

Đây là nơi thay đổi duy nhất so với bài trước. Ta sẽ trỏ địa chỉ vào MockAPI.io thay vì localhost.

File: `src/services/api.js`

```javascript
import axios from 'axios';

// 1. Dán đường link MockAPI của bạn vào đây
// LƯU Ý: Thay đổi ID dưới đây bằng ID thật của bạn vừa copy ở Bước 1.4
const MOCK_API_URL = 'https://Thay_ID_Cua_Ban_Vao_Day.mockapi.io/api/v1';

const apiClient = axios.create({
  baseURL: MOCK_API_URL, 
  headers: {
    'Content-Type': 'application/json',
  },
});

export default {
  getProducts() {
    return apiClient.get('/products');
  },
  getProduct(id) {
    return apiClient.get(`/products/${id}`);
  },
  createProduct(data) {
    return apiClient.post('/products', data);
  },
  updateProduct(id, data) {
    return apiClient.put(`/products/${id}`, data);
  },
  deleteProduct(id) {
    return apiClient.delete(`/products/${id}`);
  },
};
```

---

## Phần 4: Xây Dựng Giao Diện (Components & Views)

Phần này **HOÀN TOÀN GIỐNG** bài 16. Bạn có thể copy code từ bài trước sang. Mình sẽ tóm tắt lại code để bạn tiện theo dõi nếu làm bài này độc lập.

### 4.1 ProductCard.vue
Hiển thị từng sản phẩm.

```html
<script setup>
defineProps({ product: Object })
defineEmits(['delete'])
</script>

<template>
  <div class="card">
    <img :src="product.image" alt="SP" width="100%">
    <h3>{{ product.name }}</h3>
    <p>{{ product.price }} $</p>
    <router-link :to="`/edit/${product.id}`">Sửa</router-link>
    <button @click="$emit('delete', product.id)">Xóa</button>
  </div>
</template>

<style scoped>
.card { border: 1px solid #ccc; padding: 10px; margin-bottom: 10px; }
</style>
```

### 4.2 ProductForm.vue
Form nhập liệu dùng chung.

```html
<script setup>
import { reactive } from 'vue'
const props = defineProps({
  initialData: { type: Object, default: () => ({ name: '', price: '', image: '' }) }
})
const emit = defineEmits(['submit'])
const formData = reactive({ ...props.initialData })
</script>

<template>
  <form @submit.prevent="$emit('submit', formData)">
    <input v-model="formData.name" placeholder="Tên" required />
    <input v-model="formData.price" placeholder="Giá" required />
    <input v-model="formData.image" placeholder="Link ảnh (Tùy chọn)" />
    <button type="submit">Lưu</button>
  </form>
</template>
```

### 4.3 HomeView.vue (Trang chủ)

```html
<script setup>
import { ref, onMounted } from 'vue'
import api from '../services/api'
import ProductCard from '../components/ProductCard.vue'

const products = ref([])

onMounted(async () => {
  // Lúc này nó sẽ gọi lên MockAPI.io chứ không phải localhost
  const res = await api.getProducts()
  products.value = res.data
})

const handleRemove = async (id) => {
  if(confirm("Xóa nhé?")) {
    await api.deleteProduct(id)
    products.value = products.value.filter(p => p.id !== id)
  }
}
</script>

<template>
  <h1>Danh Sách MockAPI Online</h1>
  <div style="display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px;">
    <ProductCard 
      v-for="p in products" 
      :key="p.id" 
      :product="p" 
      @delete="handleRemove"
    />
  </div>
</template>
```

*_Lưu ý: Các view CreateView, EditView và Router setup y hệt bài 16_*

---

## Phần 5: Chạy Thử & Kiểm Tra

1.  Chạy ứng dụng:
    ```bash
    npm run dev
    ```
2.  Mở trình duyệt: `http://localhost:5173`.
3.  **Thử Thêm Mới**:
    - Bấm nút thêm, điền thông tin.
    - Sau khi lưu, hãy quay lại trang Dashboard của MockAPI.io, refresh trình duyệt, bạn sẽ thấy dữ liệu mới xuất hiện trên đó!
4.  **Thử Xóa/Sửa**:
    - Tương tác trên web local của bạn, dữ liệu trên MockAPI cloud cũng thay đổi theo.

## Tổng Kết Sự Khác Biệt

| Đặc điểm | json-server (Bài 16) | MockAPI.io (Bài 17) |
| :--- | :--- | :--- |
| **Nơi chứa dữ liệu** | File `db.json` trên máy bạn | Server của MockAPI.io (Cloud) |
| **Cài đặt** | Cần cài `json-server` (`npm i json-server`) | Không cần cài, chỉ cần đăng ký Web |
| **Môi trường** | Chỉ chạy nội mạng (Offline) | Chạy mọi nơi có Internet (Online) |
| **Phù hợp** | Dev một mình, test nhanh | Demo cho khách hàng, làm việc nhóm |

**Lời khuyên**: Khi học thì dùng `json-server` cho nhanh. Khi làm bài tập nhóm hoặc muốn demo sản phẩm mà chưa biết code Backend thật, hãy dùng `MockAPI.io`.

<template>
    <div id="container">
        <div id="Purchase-history-container">
            <menuComponent></menuComponent>
            <div id="Purchase-history-right">
                <h1>ประวัติสินค้าที่ถูกซื้อ</h1>
                <button @click="toggleSortOrder" class="sort-button">
                    {{ sortOrder === 'desc' ? 'เรียงจากใหม่ไปเก่า' : 'เรียงจากเก่าไปใหม่' }}
                </button>
                <div id="item-container">
                    <div v-for="product in sortedHistory" :key="product.productId" class="item">
                        <div class="line"></div>
                        <div class="items">
                            <input type="checkbox" v-model="product.checkbox" class="checked"
                                @change="setSelectItem(product)">
                            <img :src="product.image" alt="" style="width: 50px; height: 50px; object-fit: cover;">
                            <div class="item-1">
                                <p><span>{{ product.nameProduct }}</span></p>
                            </div>
                            <div class="item-2">
                                <p>ตัวเลือก:<span>{{ product.productTypes }}</span></p>
                                <p>จำนวน:<span>{{ product.quantity }}</span></p>
                                <p>{{ calculateLinePrice(product).toFixed(2) }} บาท</p>
                                <p><span>{{ new Date(product.createdAt).toLocaleString('th-TH') }}</span></p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="item-button">
                    <!-- <button @click="handleClick">สั่งซื้ออีกครั้ง</button> -->
                    <button @click="cancelOrder">ลบรายการประวัติสินค้า</button>
                </div>
            </div>
        </div>
    </div>
    <footerComponent></footerComponent>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            history: [],
            currentPage: 1,
            itemsPerPage: 12,
            totalItems: 0,
            shops: [],
            products: [],
            selectedItems: [],
            shopId: "",
            sortOrder: 'desc', 
        };
    },
    computed: {
        totalPages() {
            return Math.ceil(this.totalItems / this.itemsPerPage);
        },
        paginatedProducts() {
            const start = (this.currentPage - 1) * this.itemsPerPage;
            const end = start + this.itemsPerPage;
            return this.history.slice(start, end);
        },
        sortedHistory() {
            return this.history.slice().sort((a, b) => {
                const dateA = new Date(a.createdAt);
                const dateB = new Date(b.createdAt);
                return this.sortOrder === 'desc' ? dateB - dateA : dateA - dateB;
            });
        },
    },
    methods: {
        toggleSortOrder() {
            this.sortOrder = this.sortOrder === 'desc' ? 'asc' : 'desc';
        },
        totalPrice() {
            if (this.selectedItems.length > 0) {
                return this.calculateLinePrice(this.selectedItems[0]).toFixed(2);
            }
            return 0;
        },
        calculateLinePrice(product) {
            return product.price * product.quantity;
        },
        filteredProducts(shopId) {
            return this.products.filter(product => product.shopId === shopId);
        },
        async loadHistory() {
            try {
                const response = await axios.get(`http://localhost:8081/products/gethistorySeller`);
                this.history = response.data.filter(entry => entry.shopId === this.shopId);
                // this.totalItems = this.history.length;
                console.log('Loaded history:', this.history);
    
                // console.log('Total items in history:', this.totalItems);
            } catch (error) {
                console.error('Error fetching history:', error);
            }
        }, async loadshop() {
            try {
                const user = JSON.parse(localStorage.getItem('user'));
                if (!user) throw new Error('User not found in localStorage');
                const userEmail = user.email;
                if (!userEmail) throw new Error('User email not found');

                const response = await axios.get(`http://localhost:8081/shop/shops`);
                console.log(response.data);

                const shopp = Array.isArray(response.data.data)
                    ? response.data.data.filter(entry => entry.email === userEmail)
                    : [];

                if (shopp.length > 0) {
                    this.shopId = shopp[0].shopId;
                }

                console.log("this.shopId:", this.shopId);

            } catch (error) {
                console.error('Error fetching shops:', error);
            }
        }
        ,
        reorder(product) {
            console.log('Reordering product:', product);
        },
        async cancelOrder() {
            const itemsToRemove = this.selectedItems.filter(product => product.checkbox);
            try {

                for (const product of itemsToRemove) {
                    console.log('History ID to delete:', product.Historyid);
                    await axios.delete(`http://localhost:8081/products/delHistorySellrt/${product.Historyid}`);
                    console.log(`Deleted product with ID: ${product.productId}`);
                }
                await this.loadHistory();
                window.location.reload();
            } catch (error) {
                console.error('Error canceling orders:', error);
            }
        },
        gotoPage(page) {
            this.currentPage = page;
        },
        async getProducts() {
            try {
                const response = await axios.get("http://localhost:8081/products/getHistory");
                this.products = response.data;
                console.log("Fetched Products:", this.products);
            } catch (error) {
                console.error("Error fetching products:", error.response ? error.response.data : error);
            }
        },
        setSelectItem(product) {
            if (product.checkbox) {
                this.selectedItems.push(product);

            } else {
                const index = this.selectedItems.findIndex(item => item.id === product.id);
                if (index !== -1) {
                    this.selectedItems.splice(index, 1);
                }
            }
            console.log("Selected items:", this.selectedItems);
        },

    },
    async mounted() {
        this.loadshop();
        await this.getProducts();
        await this.loadHistory();

    }
};
</script>




<style scoped>
.checked {
    width: 25px;
    height: 25px;
}

#Purchase-history-container {
    width: 1200px;
    height: auto;
    min-height: 690px;
    background-color: #EBE3D5;
    display: flex;

}
.sort-button{
    width: 150px;
    height: 30px;
}
p {
    font-size: 16px;
}

#Purchase-history-right {
    padding: 20px;
    width: 920px;
    min-height: 690px;
    height: auto;
    /* เปลี่ยนจากค่า height คงที่เป็น auto */
    background-color: #F4F4F5;
    display: flex;
    flex-direction: column;
    overflow: visible;
}

.items {
    display: flex;
    gap: 10px;
    width: 100%;
}

.item-2 {
    display: flex;
    gap: 70px;
    margin-left: 70px;
    align-items: center;
}

.item-2 span {
    font-size: 16px;
}

.item-2 button {
    height: 40px;
}

.item-1 span {
    font-size: 16px;
}

.item {
    width: 94%;
    background-color: #ffffff;
    padding: 1rem 1.3rem 1rem 1.3rem;
}

.line {
    width: 100%;
    height: 2px;
    background-color: rgb(179, 172, 172);
    margin: 10px 0 10px 0;
}

.item-button {
    display: flex;
    width: 907px;
    margin-top: 10px;
    padding-right: 50px;
    justify-content: flex-end;
    gap: 2rem;
}

#container {
    height: auto;
    display: flex;
    width: 100vw;
    flex-direction: column;
    align-items: center;
}


#item-container {
    display: flex;
    flex-direction: column;
    gap: 10px;
    flex-shrink: 0;
}

.item-button :first-child {
    width: 182px;
    height: 41px;
    box-shadow: none;
    background: #fde0ae;
    border: 0.6px solid #fcdca6;
}


#Purchase-history-pagination {
    display: flex;
    margin-top: 80px;
    gap: 5px;
    width: 900px;
    justify-content: flex-end;
}

#Purchase-history-pagination button {
    width: auto;
    padding: 0px 10px 5px 10px;
    height: 20px;
}

.icon {
    font-size: 16px;
}
</style>
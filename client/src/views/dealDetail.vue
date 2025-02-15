<template>
  <main class="mt-3">
    <div class="container">
      <div class="row">
        <div class="col-md-5">
          <div
            id="carouselExampleIndicators"
            class="carousel slide carousel-dark"
            data-bs-ride="true"
          >
            <div class="carousel-indicators">
              <button
                type="button"
                data-bs-target="#carouselExampleIndicators"
                data-bs-slide-to="0"
                class="active"
                aria-current="true"
                aria-label="Slide 1"
              ></button>
              <button
                type="button"
                data-bs-target="#carouselExampleIndicators"
                data-bs-slide-to="1"
                aria-label="Slide 2"
              ></button>
              <button
                type="button"
                data-bs-target="#carouselExampleIndicators"
                data-bs-slide-to="2"
                aria-label="Slide 3"
              ></button>
            </div>
            <div class="carousel-inner">
              <div
                :class="`carousel-item ${i == 0 ? 'active' : ''}`"
                :key="i"
                v-for="(productImg, i) in productImage"
              >
                <img :src="getImageUrl(productImg)" class="d-block w-100" alt="..." />
              </div>
            </div>
            <button
              class="carousel-control-prev"
              type="button"
              data-bs-target="#carouselExampleIndicators"
              data-bs-slide="prev"
            >
              <span class="carousel-control-prev-icon" aria-hidden="true"></span>
              <span class="visually-hidden">Previous</span>
            </button>
            <button
              class="carousel-control-next"
              type="button"
              data-bs-target="#carouselExampleIndicators"
              data-bs-slide="next"
            >
              <span class="carousel-control-next-icon" aria-hidden="true"></span>
              <span class="visually-hidden">Next</span>
            </button>
          </div>
        </div>
        <div class="col-md-7">
          <div class="card shadow-sm">
            <div class="card-body">
              <h5 class="card-title">{{ productDetail.name }}</h5>
              <h5 class="card-title pt-3 pb-3 border-top">
                단가 : {{ formatCurrency(productDetail.price) }}원 | 구매 단위 :
                {{ productDetail.portion }}{{ productDetail.unit }}
              </h5>
              <p class="card-text border-top pt-3">
                <span class="badge bg-dark me-1">{{ productDetail.category }}</span>
                <span class="badge bg-red me-1"
                  >{{ productDetail.ordered }}/{{ productDetail.people }}명</span
                >
                <span class="badge bg-blue me-1">3일뒤 마감</span>
              </p>
              <p class="card-text pb-3">{{ productDetail.source }}</p>
              <p class="card-text pb-3">마포구 상암동</p>
              <div class="card-text border-top pb-3">
                <div class="row">
                  <div class="col-auto">
                    <label class="col-form-label">구매수량</label>
                  </div>
                  <div class="col-auto">
                    <div class="input-group">
                      <span
                        class="input-group-text"
                        style="cursor: pointer"
                        @click="changeCount('-')"
                        >-</span
                      >
                      <input
                        type="text"
                        class="form-control"
                        id="userPrice"
                        style="width: 50px"
                        v-model="total"
                        @keyup="calculatePrice()"
                      />
                      <span
                        class="input-group-text"
                        style="cursor: pointer"
                        @click="changeCount('+')"
                        >+</span
                      >
                    </div>
                  </div>
                </div>
              </div>
              <div class="row pt-3 pb-3 border-top">
                <div class="col-6">
                  <h3>총 상품 금액</h3>
                </div>
                <div class="col-6" style="text-align: right">
                  <h3 id="totalPrice">{{ currencyFormat(totalPrice) }}</h3>
                </div>
              </div>
              <div class="d-flex justify-content-between align-items-center">
                <div class="col-12 d-grid p-1">
                  <button type="button" class="btn btn-lg btn-danger">공구 참여</button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="row mt-5">
        <div class="col-12">
          {{ productDetail.content }}
        </div>
      </div>
    </div>
  </main>
</template>

<script setup>
import useAxios from "@/modules/axios";
import currencyFormat from "@/modules/currencyFormatter.js";
import { ref, onMounted } from "vue";
import { useRoute } from "vue-router";
const { axiosGet, axiosPost } = useAxios();

const route = useRoute();
let productId = route.query.product_id;

let productDetail = ref({}); //현재 조회중인 제품에 대한 정보
let productImage = ref([]); //현재 조회중인 제품의 이미지 명(XXX.PNG)
let total = ref(1); //사용자가 구매하고자 하는 물건의 개수
let totalPrice = ref(0); //사용자의 총 구매 금액

//사용자가 물건 개수를 입력하지 않았을 때, 1로 바꾸고 가격도 다시 계산
onMounted(() => {
  const userPrice = document.getElementById("userPrice");
  userPrice.addEventListener("focusout", checkData);

  function checkData(event) {
    if (total.value < 1) {
      total.value = 1;
      totalPrice.value = productDetail.value.price;
    }
  }
});

//total 계산
function changeCount(cnt = "") {
  cnt === "+" ? total.value++ : total.value--;
  if (total.value < 1) total.value = 1;
  calculatePrice();
}

//totalPrice 계산
function calculatePrice() {
  totalPrice.value = productDetail.value.price * total.value;
}

//금액을 #,### 형태로 포맷팅한다.
function formatCurrency(value) {
  return currencyFormat(value);
}

//제품 상세 쿼리에 대한 콜백함수
const saveDetail = function (respData) {
  productDetail.value = respData[0];
  totalPrice.value = productDetail.value.price;
};

//제품 이미지 쿼리에 대한 콜백함수
const saveImage = function (respData) {
  productImage.value = respData.result.map((x) => x.path);
  console.log(productImage.value);
};

//제품 상세 쿼리
function getProductDetail() {
  axiosGet(`/product/${productId}`, saveDetail);
}

//제품 이미지 쿼리
function getProductImage() {
  axiosGet(`/image/${productId}`, saveImage);
}

//이미지를 실제 경로에서 가져온다.
const getImageUrl = (name) => {
  return `https://gongu-image.s3.ap-northeast-2.amazonaws.com/${name}`;
};

//onCreated
getProductDetail();
getProductImage();
</script>

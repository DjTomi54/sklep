<!doctype html>
<html lang="pl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Deliro</title>
  <style>
    :root{--accent:#f0f0f0;--muted:#f0f0f0}
    *{box-sizing:border-box}
    body{font-family:Inter,system-ui,Arial,sans-serif;margin:0;background:#242626;color:#000000}
    .container{max-width:1100px;margin:24px auto;padding:16px}
    header{display:flex;justify-content:space-between;align-items:center;margin-bottom:16px;flex-wrap:wrap;gap:8px}
    h1{margin:0;font-size:20px}
    .search{display:flex;gap:8px;flex-wrap:wrap}
    input,select,button{padding:8px;border:1px solid #137858;border-radius:8px}
    button.primary{background:var(--accent);color:white;border:none;cursor:pointer}
    .grid{display:grid;grid-template-columns:1fr 320px;gap:18px}
    .products{display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:12px}
    .card{background:white;border-radius:10px;overflow:hidden;box-shadow:0 6px 18px rgba(15,23,42,0.06)}
    .card img{width:100%;height:150px;object-fit:cover;display:block}
    .card .body{padding:10px}
    .price{font-weight:700}
    aside{background:white;padding:12px;border-radius:10px;box-shadow:0 6px 18px rgba(15,23,42,0.04)}
    .cart-item{display:flex;justify-content:space-between;gap:8px;padding:8px 0;border-bottom:1px dashed #a5d9cf}
    .small{font-size:13px;color:var(--muted)}
    footer{margin-top:18px;text-align:center;color:var(--muted);font-size:13px}
    .modal{position:fixed;inset:0;background:rgba(2,6,23,0.5);display:flex;align-items:center;justify-content:center;padding:16px;z-index:100}
    .modal .panel{background:white;border-radius:10px;max-width:760px;width:100%;overflow:auto;max-height:90vh;padding:16px;position:relative}
    .modal h3{margin-top:0}
    .close-btn{position:absolute;top:8px;right:8px;cursor:pointer;background:#fa0202;border:none;border-radius:6px;padding:4px 8px}
    @media(max-width:880px){.grid{grid-template-columns:1fr}}
  </style>
</head>
<body>
  <div class="container">
    <header>
      <div>
        <h1>Deliro</h1>
        <div class="small">Szybko, prosto i wygodnie</div>
      </div>
      <div class="search">
        <input id="q" placeholder="Szukaj..." />
        <select id="sort">
          <option value="popular">Najpopularniejsze</option>
          <option value="price_asc">Cena rosnąco</option>
          <option value="price_desc">Cena malejąco</option>
        </select>
        <button id="open-cart" class="primary">Koszyk (<span id="cart-count">0</span>)</button>
      </div>
    </header>

    <main class="grid">
      <section>
        <div style="margin-bottom:12px;font-weight:600">Produkty</div>
        <div id="products" class="products"></div>
      </section>
      <aside>
        <h3>Promocja</h3>
        <p class="small">Darmowa wysyłka od 100 zł!</p>

        <p class="medium"> Szanowni państwo, w tej chwili usługa "BLIK" oraz "Przelewy 24" są nieaktywne.
        Zapraszamy do płatności poprzez "PayPal" lub kartą :) 
        Za utrudnienia przepraszamy.</p>
      </aside>
    </main>

    <footer>© 2025 DELIRO</footer>
  </div>

  <div id="modal" class="modal" style="display:none">
    <div class="panel">
      <button onclick="closeModal()" class="close-btn">✖</button>
      <div id="modal-body"></div>
      <div id="paypal-button-container" style="margin-top:20px"></div>
    </div>
  </div>

  <!-- PayPal SDK -->
  <script src="https://www.paypal.com/sdk/js?client-id=AQykTWh9w8PJV3j5k5ARmfWbHltq88zLlhriIxwuWG3ol7V1sbWhNBn_isWYLaB7ObgJAGuq2vv1t3e-&currency=PLN"></script>

  <script>
    const products=[
      {id:1,title:"Koszulka Minimal",price:0.30,img:"https://picsum.photos/seed/p1/600/400",description:"Bawełniana koszulka, krój regular."},
      {id:2,title:"Kubek Poranna kawa",price:29.99,img:"https://picsum.photos/seed/p2/600/400",description:"Ceramiczny kubek 330ml."},
      {id:3,title:"Torba płócienna",price:39.50,img:"https://picsum.photos/seed/p3/600/400",description:"Wytrzymała torba na zakupy."},
      {id:4,title:"Plakat A3",price:19.00,img:"https://picsum.photos/seed/p4/600/400",description:"Plakat drukowany na matowym papierze."}
    ];

    let cart=JSON.parse(localStorage.getItem('cart')||'[]');

    function renderProducts(){
      const q=document.getElementById('q').value.toLowerCase();
      const sort=document.getElementById('sort').value;
      let list=products.filter(p=>p.title.toLowerCase().includes(q)||p.description.toLowerCase().includes(q));
      if(sort==='price_asc') list.sort((a,b)=>a.price-b.price);
      if(sort==='price_desc') list.sort((a,b)=>b.price-a.price);
      const box=document.getElementById('products');
      box.innerHTML=list.map(p=>`
        <div class="card">
          <img src="${p.img}" alt="${p.title}" />
          <div class="body">
            <div><strong>${p.title}</strong></div>
            <div class="small">${p.description}</div>
            <div style="margin-top:6px;display:flex;justify-content:space-between;align-items:center">
              <span class="price">${p.price.toFixed(2)} zł</span>
              <button onclick='addToCart(${p.id})' class="primary">Do koszyka</button>
            </div>
          </div>
        </div>`).join('');
    }

    function addToCart(id){
      const found=cart.find(i=>i.id===id);
      if(found){found.qty++;}else{
        const prod=products.find(p=>p.id===id);
        cart.push({...prod,qty:1});
      }
      saveCart();
      updateCartCount();
    }

    function removeFromCart(id){
      cart = cart.filter(item => item.id !== id);
      saveCart();
      updateCartCount();
      openCart();
    }

    function saveCart(){localStorage.setItem('cart',JSON.stringify(cart));}
    function updateCartCount(){document.getElementById('cart-count').innerText=cart.reduce((s,i)=>s+i.qty,0);}

    function renderPayPalButton(total){
      if(typeof paypal==='undefined') return;
      document.getElementById('paypal-button-container').innerHTML='';
      paypal.Buttons({
        createOrder: function(data, actions) {
          return actions.order.create({purchase_units: [{amount:{value: total}}]});
        },
        onApprove: function(data, actions) {
          return actions.order.capture().then(function(details){
            alert('Dziękujemy za zakupy, ' + details.payer.name.given_name + '!');
            cart=[]; saveCart(); updateCartCount(); closeModal();
          });
        }
      }).render('#paypal-button-container');
    }

    function openCart(){
      const total = cart.reduce((s,i)=>s+i.qty*i.price,0).toFixed(2);
      const html = cart.length ? cart.map(i => `
        <div class='cart-item'>
          <div>${i.title} x${i.qty}</div>
          <div>
            ${(i.qty*i.price).toFixed(2)} zł
            <button onclick="removeFromCart(${i.id})" style="margin-left:8px;">Usuń</button>
          </div>
        </div>
      `).join('') : "Koszyk pusty";

      document.getElementById('modal-body').innerHTML = `<h3>Twój koszyk</h3>${html}`;
      document.getElementById('modal').style.display='flex';

      if(cart.length) renderPayPalButton(total);
      else document.getElementById('paypal-button-container').innerHTML='';
    }

    function closeModal(){document.getElementById('modal').style.display='none';}

    document.addEventListener('DOMContentLoaded', function(){
      document.getElementById('open-cart').onclick=openCart;
      document.getElementById('q').oninput=renderProducts;
      document.getElementById('sort').onchange=renderProducts;
      renderProducts();
      updateCartCount();
    });
  </script>
</body>
</html>

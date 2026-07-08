<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kinosaki Onsen Travel Guide</title>
    
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.2/css/all.min.css">
    
    <style>
        body, html { margin: 0; padding: 0; height: 100%; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; background-color: #f7f9fa; }
        #app { display: flex; height: 100vh; }
        
        /* Sidebar Styles */
        #sidebar { width: 380px; overflow-y: auto; padding: 20px; border-right: 1px solid #e1e4e6; background: #ffffff; box-sizing: border-box; display: flex; flex-direction: column; }
        h1 { font-size: 22px; color: #1a202c; margin-top: 0; margin-bottom: 5px; font-weight: 700; }
        .subtitle { font-size: 14px; color: #718096; margin-bottom: 20px; }
        
        /* Map Layout */
        #map { flex-grow: 1; height: 100%; }

        /* Mobile Optimization */
        @media (max-width: 768px) {
            #app { flex-direction: column-reverse; }
            #sidebar { width: 100%; height: 45%; border-right: none; border-top: 1px solid #e1e4e6; padding: 15px; }
            #map { height: 55%; width: 100%; }
        }

        /* Category Separators */
        .category-title { font-size: 13px; text-transform: uppercase; letter-spacing: 0.05em; color: #a0aec0; margin: 15px 0 8px 0; font-weight: 600; border-bottom: 1px solid #edf2f7; padding-bottom: 4px; }
        
        /* Interactive List Items */
        .location-item { display: flex; align-items: center; padding: 12px 10px; border-radius: 8px; cursor: pointer; margin-bottom: 5px; transition: background 0.2s ease, transform 0.1s ease; -webkit-tap-highlight-color: transparent; }
        .location-item:hover { background: #f7f9fa; }
        .location-item:active { transform: scale(0.98); background: #edf2f7; }
        .location-item-icon { width: 32px; height: 32px; border-radius: 50%; display: flex; align-items: center; justify-content: center; margin-right: 12px; font-size: 14px; flex-shrink: 0; }
        .location-item-info { flex-grow: 1; }
        .location-item-name { font-size: 15px; font-weight: 600; color: #2d3748; margin: 0 0 2px 0; }
        .location-item-desc { font-size: 12px; color: #718096; margin: 0; }

	/* Styles bộ lọc */
        #filters { display: flex; flex-wrap: wrap; gap: 7px; margin-bottom: 15px; }
        .pill { font-size: 12px; padding: 6px 12px; border-radius: 999px; background: #edf2f7; color: #4a5568; cursor: pointer; transition: .2s; border: 1px solid #e2e8f0; }
        .pill.active { background: #3182CE; color: white; border-color: #3182CE; }

        .category-title { font-size: 12px; text-transform: uppercase; color: #a0aec0; margin: 15px 0 8px 0; font-weight: 700; border-bottom: 1px solid #edf2f7; }
        .location-item { display: flex; align-items: center; padding: 10px; border-radius: 8px; cursor: pointer; border: 1px solid transparent; }
        .location-item:hover { background: #f7f9fa; }
        .location-item-icon { width: 30px; height: 30px; border-radius: 50%; display: flex; align-items: center; justify-content: center; margin-right: 12px; font-size: 12px; }

        /* Custom Map Marker Icons CSS */
        .map-custom-marker { background: #ffffff; border-radius: 50%; text-align: center; display: flex; align-items: center; justify-content: center; box-shadow: 0 3px 8px rgba(0,0,0,0.24); border: 2px solid #ffffff; width: 32px !important; height: 32px !important; }
        .map-custom-marker i { font-size: 14px; }
        
        /* POPUP DETAILS CARD */
        .popup-card { width: 280px; font-family: sans-serif; max-height: 400px; overflow-y: auto; padding-right: 5px; }
        .popup-img { width: 100%; height: 130px; object-fit: cover; border-radius: 6px; margin-bottom: 8px; background: #edf2f7; }
        .popup-title { font-weight: 700; font-size: 16px; margin-bottom: 6px; color: #1a202c; border-bottom: 1px solid #edf2f7; padding-bottom: 4px; }
        .popup-meta { font-size: 12px; color: #4a5568; margin-bottom: 4px; display: flex; align-items: flex-start; gap: 6px; line-height: 1.4; }
        .popup-meta i { color: #718096; width: 14px; text-align: center; margin-top: 2px; }
        .popup-section-title { font-size: 11px; font-weight: 700; color: #a0aec0; text-transform: uppercase; margin-top: 8px; margin-bottom: 4px; letter-spacing: 0.05em; border-bottom: 1px dashed #edf2f7; padding-bottom: 2px; }
        
        /* ĐỊNH DẠNG NÚT BẤM GOOGLE MAPS */
        .gmaps-btn { display: flex; align-items: center; justify-content: center; gap: 8px; background-color: #2da44e; color: white !important; text-decoration: none !important; font-size: 13px; font-weight: 600; padding: 8px 12px; border-radius: 6px; margin-top: 12px; text-align: center; transition: background 0.2s; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
        .gmaps-btn:hover { background-color: #2c974b; }
        .gmaps-btn i { font-size: 14px; }
    </style>
</head>
<body>

<div id="app">
    <div id="sidebar">
        <h1>Kinosaki Onsen</h1>
        <div class="subtitle">Tap any location below to view details and photos</div>
        <div id="list-container"></div>
    </div>
    <div id="map"></div>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
    // Tự động đặt tâm bản đồ (Center) ngay tại Ryokufu kaku [35.62445, 134.81135] với độ zoom 17 gần hơn để khách dễ nhìn thấy điểm xuất phát
    const map = L.map('map', { zoomControl: true, tap: false }).setView([35.62445, 134.81135], 17);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

    // =========================================================================
    // KHU VỰC DỮ LIỆU ĐỊA ĐIỂM
    // =========================================================================
    const dataset = {
        // Đưa địa điểm làm việc của bạn lên vị trí đầu tiên
        "Starting Point": [
            {
                name: "Ryokufu kaku (緑風閣)", lat: 35.6254238, lng: 134.8121012, icon: "fa-hotel", color: "#E53E3E", // Màu đỏ nổi bật
                image: "", google_maps_link: "https://maps.app.goo.gl/Bd2K2vFtW99rzAxu6",
                recommended_by: "",
                highlight: "Welcome to Ryokufu kaku!<br><br>",
                opening_hours: "Check-in from 15:00", dayoff: "July: 1st, 7th, 8th, 9th, 14th, 15th, 21st, 28th"
            }
        ],
        
        "Hot Springs (Sotoyu)": [
           
            { 
                name: "Jizou-yu (地蔵湯)", lat: 35.6267329, lng: 134.8126665, icon: "fa-hot-tub-person", color: "#3182CE", image: "", google_maps_link: "https://maps.app.goo.gl/UFhtH5dm4FXLUHG29", 
                recommended_by: "",
                highlight: "Shaped like a traditional Japanese lantern. Very popular among local families with a retro indoor bath vibe.",
                opening_hours: "07:00 - 23:00", dayoff: "Monday" 
            },
            { 
                name: "Yanagi-yu (柳湯)", lat: 35.6264888, lng: 134.8103239, icon: "fa-hot-tub-person", color: "#3182CE", image: "", google_maps_link: "https://maps.app.goo.gl/qhcUuJowoJzwDhFo9", 
                recommended_by: "",
                highlight: "Named after the willow trees lining the river front. Cozy and rustic Cypress wood (hinoki) interior.",
                opening_hours: "15:00 - 23:00", dayoff: "Thursday" 
            },
            { 
                name: "Ichino-yu (一の湯)", lat: 35.6261735, lng: 134.8094847, icon: "fa-hot-tub-person", color: "#3182CE", image: "", google_maps_link: "https://maps.google.com/?q=Ichino-yu+Kinosaki", 
                recommended_by: "",
                highlight: "Known as the 'Number One Hot Spring'. Features an incredible cave bath carved directly out of natural rock.",
                opening_hours: "07:00 - 23:00", dayoff: "Wednesday" 
            },
            { 
                name: "Gosho-no-yu (御所の湯)", lat: 35.6259683, lng: 134.8073577, icon: "fa-hot-tub-person", color: "#3182CE", image: "", google_maps_link: "https://maps.app.goo.gl/PVEM6EdXqjT4XPM97", 
                recommended_by: "",
                highlight: "The 'Imperial Bath' built to look like a Kyoto palace.<br><br>Beautiful outdoor waterfall views while you soak.",
                opening_hours: "07:00 - 23:00", dayoff: "Thursday" 
            },
            { 
                name: "Mandara-yu (まんだら湯)", lat: 35.6246076, lng: 134.8055821, icon: "fa-hot-tub-person", color: "#3182CE", image: "", google_maps_link: "https://maps.google.com/?q=Mandara-yu+Kinosaki", 
                recommended_by: "",
                highlight: "Features lovely open-air ceramic barrel baths with peaceful mountain views tucked away in a quiet alley.",
                opening_hours: "15:00 - 23:00", dayoff: "Wednesday" 
            },
            { 
                name: "Kouno-yu (鴻の湯)", lat: 35.62630395533045, lng: 134.80448588042717, icon: "fa-hot-tub-person", color: "#3182CE", image: "", google_maps_link: "https://maps.google.com/?q=Kouno-yu+Kinosaki", 
                recommended_by: "",
                highlight: "The oldest public bath in Kinosaki. Legend says an injured Oriental White Stork cured its leg here. Great garden views.",
                opening_hours: "07:00 - 23:00", dayoff: "Tuesday"
            }
        ],
        
        "Lunch Spots": [
            { 
                name: "Okesho Fish Market & Restaurant", lat: 35.6251014, lng: 134.8127251, icon: "fa-utensils", color: "#DD6B20",
                image: "https://i.imgur.com/h4sN8P1.jpeg",
                google_maps_link: "https://maps.app.goo.gl/QUeawB338L7y2y8b8", 
                recommended_by: "Ohshima (part-time staff)",
                highlight: "Kaisendon.<br> Because they use fresh fish caught that very morning, the colors are beautiful and the meat has a delightfully firm, plump texture. Plus, since the daily catch changes based on what's available, the wide variety of toppings is part of the charm!",
                opening_hours: "11:00-19:00", dayoff: "Irregular holidays",
                price_range: "800￥ ~ 3.300￥", credit_card: "Yes",
                dine_in: "Dine-in", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken",
                peak_hours: "11:00 ~ 13:30", avg_waiting_time: "0-15 minutes", reservation: "Recommended"
            },
            { 
                name: "Hoshi Soba", lat: 35.6245433, lng: 134.8128033, icon: "fa-utensils", color: "#DD6B20",
                image: "https://i.imgur.com/V8jwxFe.jpeg",
                google_maps_link: "https://maps.app.goo.gl/UJSdZgWwDf6bPY8T8", 
                recommended_by: "Ms.Ushine (Staff)",
                highlight: "Sara Soba.<br> We highly recommend Hoshi Soba if you're looking for a fantastic lunch spot! The restaurant is modern, clean, and very welcoming. Their authentic Izushi-style Sara Soba (Dish Soba) is a must-try; the noodles are freshly made in-house and have a wonderful chewy texture with a rich buckwheat aroma.",
                opening_hours: "11:00–15:00", dayoff: "Irregular holidays",
                price_range: "1.100￥ ~ 1.700￥", credit_card: "Yes",
                dine_in: "Dine-in", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken",
                peak_hours: "~11:00", avg_waiting_time: "None", reservation: "Not needed"
            },
            { 
                name: "Yamayoshi", lat: 35.6241159, lng: 134.8130899, icon: "fa-utensils", color: "#DD6B20",
                image: "https://i.imgur.com/GObBqmM.jpeg", google_maps_link: "https://maps.app.goo.gl/jnDpF8UpP9JgZV5R8", 
                recommended_by: "Mr.Nakata (Manager)",
                highlight: "Tokujo Kanizushi Teisyoku. <br> We highly recommend Oshokujidokoro Yamayoshi near Kinosaki Onsen Station for a fantastic seafood lunch! You absolutely must try their Tokujo Kanizushi Teishoku (Premium Crab Sushi Set). The crab is incredibly fresh, sweet, and melts in your mouth. Definitely give it a try!",
                opening_hours: "11:00–15:00", dayoff: "Thursday",
                price_range: "800￥ ~ 3.400￥", credit_card: "Yes",
                dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken",
                peak_hours: "~12:00", avg_waiting_time: "10min ~ 30min", reservation: "Not needed"
            }
        ],
        
        "Lunch & Dinner": [
            { 
                name: "UnaKyo", lat: 35.627352027127344, lng: 134.8120488265592, icon: "fa-bowl-rice", color: "#DD6B20",
                image: "https://i.imgur.com/xTQgsUi.jpeg", google_maps_link: "https://maps.app.goo.gl/yAJXjejAaAAir7Ku5", 
                recommended_by: "Mr.Yamamoto (Leader)",
                highlight: "Superior grilled eel on rice. <br>We highly recommend Kinosaki Unagi-dokoro Unakyo for an amazing dining experience! You must try their Hitsumabushi or premium eel dishes sourced from Lake Hamana. The eel is exceptionally fresh—they even show it to you live before preparing it to perfection. It offers incredible value, top-tier flavor, and wonderful hospitality.",
                opening_hours: "11:00–14:00, 18:00–22:00", dayoff: "Saturday, Sunday",
                price_range: "1.198¥ ~ 3.980¥", credit_card: "Yes",
                dine_in: "Dine-in", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken",
                peak_hours: "19:00 ~ 20:00", avg_waiting_time: "10 minutes", reservation: "Recommended"
            },
            { 
                name: "Kamoramen Tanuki", lat: 35.6253202, lng: 134.8080229, icon: "fa-bowl-food", color: "#DD6B20",
                image: "https://i.imgur.com/WF8u5ps.jpeg", google_maps_link: "https://maps.app.goo.gl/fHLSFyJunn8fPdnf8", 
                recommended_by: "Ms.Kurita (Part-time Staff)",
                highlight: "Shoyu ramen. <br>We highly recommend Kamo Ramen Tanuki if you're looking for a truly unique meal in Kinosaki Onsen! Tucked away in Kiyamachi Kojii alley, this hidden gem completely changes the game with its deep, comforting Duck Shoyu Ramen. It’s a tiny, bustling shop that easily sells out early during the day, but they also open for dinner on select evenings—making it the perfect, unforgettable spot to hit up right at opening!",
                opening_hours: "11:00–15:00, 17:00–19:00", dayoff: "Irregular holidays",
                price_range: "950¥ ~ 2.250¥", credit_card: "Yes",
                dine_in: "Dine-in", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken",
                peak_hours: "14:00~", avg_waiting_time: "~10 minutes", reservation: "Recommended"
            }
        ],

        "Sweets & Snacks": [
            {
                name: "Gyusho Ueda Kinosaki Store", lat: 35.6253287, lng: 134.8127000, icon: "fa-stroopwafel", color: "#673AB7",
                image: "https://i.imgur.com/RY7u5ID.jpeg", google_maps_link: "https://maps.app.goo.gl/aGFpCyReXBeuk9pm8",
                recommended_by: "Mr.Nakata (Manager)",
                highlight: "Tajima Steak Skewers.<br> Their freshly grilled Tajima Beef Steak Skewers are melt-in-your-mouth tender, and their made-to-order Minced Meat Cutlet is unbelievably crispy and juicy. It’s a popular butcher shop with limited standing space—perfect for a premium, delicious bite while exploring the hot spring town!",
                opening_hours: "10:00–17:30", dayoff: "Irregular holidays",
                price_range: "250¥ ~ 1.800¥", credit_card: "Yes",
                dine_in: "Dine-in / Takeout", group_size: "Solo diners / couples / large groups", english_support: "English menu / English spoken",
                peak_hours: "12:00~", avg_waiting_time: "~5 minutes", reservation: "Not needed"
            },
            {
                name: "Chikara Mochi", lat: 35.625797, lng: 134.808165, icon: "fa-ice-cream", color: "#673AB7",
                image: "https://i.imgur.com/NX6pfiY.jpeg",
                google_maps_link: "https://maps.app.goo.gl/yMZR16CtVnfXRGKM8",
                recommended_by: "Mr.Nakata (Manager)",
                highlight: "Vanilla Ice Cream. <br>We highly recommend Chikara Mochi for a nostalgic treat in Kinosaki Onsen! While they are known for noodles, their Vanilla Soft Serve Ice Cream is a hidden superstar—pure, old-school flavor and served super fast. It's the perfect refreshing treat to grab on the go while strolling between hot springs or viewing evening cherry blossoms!",
                opening_hours: "11:00–22:00", dayoff: "Thursday",
                price_range: "550¥ ~ 950¥", credit_card: "Yes",
                dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken",
                peak_hours: "None", avg_waiting_time: "None", reservation: "Not needed"
            },
            {
                name: "Katashima Gluten Free", lat: 35.6255019, lng: 134.8079826, icon: "fa-cookie-bite", color: "#673AB7",
                image: "https://i.imgur.com/Xh5LLEI.jpeg",
                google_maps_link: "https://maps.app.goo.gl/56tmzuE79pTspEAV9",
                recommended_by: "Ms.Ushine (Staff)",
                highlight: "Milk & Coconut Canelé; Caramel Canelé. <br>We highly recommend Katashima Seikodo in Kinosaki Onsen for their amazing gluten-free rice flour canelés! The Milk Coconut flavor offers a delightful creamy, tropical sweetness, while the Caramel is a rich, crunchy-outside, chewy-inside classic. It’s a trendy, popular little shop in Kiyamachi Kojii alley—perfect for a sweet treat on the go!",
                opening_hours: "10:00–17:00", dayoff: "Wednesday, Thursday",
                price_range: "180¥ ~ 750¥", credit_card: "Yes / No Cash",
                dine_in: "Takeout only", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken / None",
                peak_hours: "13:00~", avg_waiting_time: "-", reservation: "Not needed"
            },
            {
                name: "Hasegawa Tofu Shop", lat: 35.6252311, lng: 134.8118695, icon: "fa-cheese", color: "#673AB7",
                image: "https://i.imgur.com/LXzmqYj.jpeg",
                google_maps_link: "https://maps.app.goo.gl/HzgmGZbBR724Mcx37",
                recommended_by: "Ms.Ushine, Ms.Han (Staff)",
                highlight: "Tofu Donuts. <br>You absolutely have to check out Hasegawa Tofu Shop for a healthy snack break in Kinosaki Onsen! Their freshly made Tofu Donuts are wonderfully light, fluffy, and perfectly sweet. If you visit during the winter, you can even soak your feet in their cozy indoor footbath while enjoying your treats—making it a perfect hidden gem to warm up and recharge during your hot spring stroll!",
                opening_hours: "10:00–18:00", dayoff: "Tuesday, Thursday",
                price_range: "130¥ ~ 650¥", credit_card: "Cash Only",
                dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English spoken",
                peak_hours: "10:00 (winter)", avg_waiting_time: "None", reservation: "Not needed"
            },
            {
                name: "Egg Specialty Motosue Kinosaki Sohonke", lat: 35.6242405, lng: 134.8132173, icon: "fa-egg", color: "#673AB7",
                image: "https://i.imgur.com/Ddem1Tr.jpeg",
                google_maps_link: "https://maps.app.goo.gl/Kk9T5Wnu9ccb4ftw7",
                recommended_by: "Han (Full-time staff)",
                highlight: "Chocolate Tamago Pan <br>Their Chocolate Tamago Pan is an absolute dream—combining their signature jiggly, melt-in-your-mouth egg bread with a rich, velvety chocolate twist. It's perfectly sweet, deeply flavorful, and makes for the ultimate premium snack to grab as soon as you step off the train!",
                opening_hours: "10:00 ~ 17:00", dayoff: "Irregular holidays",
                price_range: "520￥ ~ 1900￥", credit_card: "Yes",
                dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English spoken",
                peak_hours: "10:00 ~ 14:00", avg_waiting_time: "0-10 minutes", reservation: "Not needed"
            },
            {
                name: "Koori to Kani", lat: 35.6241121, lng: 134.8130168, icon: "fa-crab", color: "#673AB7",
                image: "https://i.imgur.com/bRMkdta.jpeg",
                google_maps_link: "https://maps.app.goo.gl/deabhFiu3hyJHbkD6",
                recommended_by: "Ms.Huong (Staff)",
                highlight: "Koura-yaki (Grilled crab paste in shell).<br> Their signature Koura-yaki is a must-try crab doria baked inside a real crab shell. It is packed with sweet, tender crab meat and rich crab miso (kani miso), then topped with golden, melted cheese. It's the ultimate quick, premium bite for crab lovers!",
                opening_hours: "10:00–17:00", dayoff: "Irregular holidays",
                price_range: "1.480¥ ~ 2.900¥", credit_card: "Yes",
                dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken",
                peak_hours: "13:00 ~ 15:00", avg_waiting_time: "~5 minutes", reservation: "Not needed"
            },
            {
                name: "Ukawa", lat: 35.6241059, lng: 134.8130152, icon: "fa-bread-slice", color: "#673AB7",
                image: "https://i.imgur.com/WkgDyM4.jpeg",
                google_maps_link: "",
                recommended_by: "Ms.Okamoto (Staff)",
                highlight: "Kinako-pan. <br>Their signature Kinako Donut is a total crowd-pleaser—crispy on the outside, incredibly fluffy on the inside, and coated in fragrant black soybean kinako powder. It’s the ultimate sweet, comforting snack to grab for your hot spring stroll!",
                opening_hours: "10:00–17:00", dayoff: "Irregular holidays",
                price_range: "300¥ ~ 1.200¥", credit_card: "Yes",
                dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken",
                peak_hours: "13:00 ~ 15:00", avg_waiting_time: "None", reservation: "Not needed"
            }
        ],

        "Cafes & Terraces": [
            {
                name: "Chaya", lat: 35.6256473, lng: 134.8042581, icon: "fa-mug-hot", color: "#795548",
                image: "https://i.imgur.com/WfsRjr8.jpeg",
                google_maps_link: "https://maps.app.goo.gl/vbctb8fWNfGMcdnU7",
                recommended_by: "Han (Staff)",
                highlight: "Onsen Egg (DIY Onsen tamago). <br>I highly recommend trying the DIY Onsen Tamago experience at Chaya! It’s a super fun, unique interactive snack break while exploring Kinosaki Onsen. You just buy fresh raw eggs at the shop, drop them into the natural hot spring source right outside, and wait about 9 minutes while soaking your feet in the nearby footbath. It’s a cozy, bustling little spot—definitely check it out!",
                opening_hours: "09:30–17:30", dayoff: "Thursday",
                price_range: "350¥ ~ 1.000¥", credit_card: "Yes",
                dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken",
                peak_hours: "11:00 / 15:00", avg_waiting_time: "None", reservation: "Not needed"
            },
            {
                name: "Book Cafe Un", lat: 35.6248752, lng: 134.8127426, icon: "fa-book-open", color: "#795548",
                image: "https://i.imgur.com/D6rwJFH.jpeg",
                google_maps_link: "https://maps.app.goo.gl/QuKAgbgpi3636naW8",
                recommended_by: "Mr.Matsuo",
                highlight: "Butter Dorayaki. <br>This is a cozy, relaxing book cafe produced by Nishimuraya, a long-established traditional ryokan. While their signature item is the Fresh Butter Dorayaki, my personal top recommendations are the 'Cheese Chocolate' and 'Salt Milk' dorayaki! The balance between sweet and savory is absolutely perfect.",
                opening_hours: "10:00–17:00", dayoff: "Thursday",
                price_range: "330¥ ~ 600¥", credit_card: "Yes",
                dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken ",
                peak_hours: "13:00 ~ 15:00", avg_waiting_time: "None", reservation: "Not needed"
            },
		{
        name: "Kiyamachi Koji Kinosaki Vinegar Shop", lat: 35.6255186, lng: 134.8079662, icon: "fa-wine-bottle", color: "#795548",
        image: "https://i.imgur.com/9vDLc4q.jpeg", google_maps_link: "https://maps.app.goo.gl/r7vBEWCXB5BNa9fT7",
        recommended_by: "Ms.Ushine (Staff)",
        highlight: "Pink Grapefruit Vinegar (Soda style)<br>Their Pink Grapefruit Soda is a total standout—the perfect balance of tangy, citrusy grapefruit flavor with a crisp, effervescent kick that instantly cools down and recharges your body. It's a delightful, healthy drink to enjoy while strolling around town!",
        opening_hours: "10:00–17:00", dayoff: "Thursday",
        price_range: "330¥ ~ 440¥", credit_card: "Yes",
        dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken ",
        peak_hours: "None", avg_waiting_time: "None", reservation: "Not needed"
    },
	{
        name: "Kinosaki Coffee Miharashi Terrace Cafe", lat: 35.6225840, lng: 134.7972662, icon: "fa-utensils", color: "#795548",
        image: "https://i.imgur.com/3jqFlHV.jpeg", google_maps_link: "https://maps.app.goo.gl/yYdQjCfjJtmncTdf7",
        recommended_by: "Mr.Nakata (Manager)",
        highlight: "Hot Dog Set<br>Their Tajima Beef Hot Dog Set is a real treat—featuring a juicy, flavorful sausage made from premium local Tajima beef, served in a perfectly toasted bun. Paired with their freshly brewed blend coffee, it’s the ultimate savory combo to enjoy while soaking in the breathtaking panoramic views of the town and sea!",
        opening_hours: "10:00–16:00", dayoff: "Thursday",
        price_range: "380¥ ~ 1.300¥", credit_card: "Yes",
        dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken ",
        peak_hours: "10:00–15:00", avg_waiting_time: "5 min", reservation: "Not needed"
    },
	{
        name: "PARADI", lat: 35.6249458, lng: 134.8074298, icon: "fa-bread-slice", color: "#795548",
        image: "https://i.imgur.com/MTw45c9.jpeg", google_maps_link: "https://maps.app.goo.gl/Y1FrgFe4pzSJkDrR9",
        recommended_by: "Mr.Nakata (Manager)",
        highlight: "Croissant<br>Their signature Croissant is an absolute masterpiece—unbelievably flaky, crispy, and shattered with a rich, buttery aroma on the outside, while remaining beautifully light and airy on the inside. Grab one with an iced coffee and sit at their unique bridge seating over the water for the ultimate French-cafe vibe right in the heart of the hot spring town!",
        opening_hours: "09:00–18:00", dayoff: "Thursday",
        price_range: "500¥ ~ 1.750¥", credit_card: "Yes",
        dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken ",
        peak_hours: "09:00–11:00", avg_waiting_time: "None", reservation: "Not needed"
    },
	{
        name: "Oimo to Canelé Kinosaki Ashiyu Cafe", lat: 35.6242293, lng: 134.8128120, icon: "fa-cookie-bite", color: "#795548",
        image: "https://i.imgur.com/Mzr6a3d.jpeg", google_maps_link: "https://maps.app.goo.gl/7DSmepB516c291N48",
        recommended_by: "Ms.Han",
        highlight: "Baked Sweet Potato Canelé Soft Serve<br>If you're a fan of sweet treats, Oimo to Canelé Kinosaki Ashiyu Cafe is your next stop! Their Baked Sweet Potato Canelé Soft Serve is a total masterpiece, featuring velvety ice cream paired with sweet roasted potato and topped with a crunchy, palm-sized canelé. It's the ultimate decadent dessert to enjoy while soaking your feet in their indoor footbath!",
        opening_hours: "09:00–18:00", dayoff: "Irregular holidays",
        price_range: "200¥ ~ 2.500¥", credit_card: "Yes",
        dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken ",
        peak_hours: "13:00–17:00", avg_waiting_time: "None", reservation: "Not needed"
    },
    {
        name: "Kinosaki Pudding Specialty Shop - Kiman", lat: 35.6262806, lng: 134.8115017, icon: "fa-ice-cream", color: "#795548",
        image: "https://i.imgur.com/V0W4RDS.jpeg", google_maps_link: "https://maps.app.goo.gl/GkoqWaMj1xxC53SN7",
        recommended_by: "Ms.Huong",
        highlight: "Assorted Puddings<br>Their signature Premium Pudding is an absolute masterpiece—so incredibly soft and velvety that it practically melts the moment it hits your tongue. For the ultimate experience, try their popular assortment platter to taste and compare their classic egg pudding, rich crème brûlée, and premium flavors all at once!",
        opening_hours: "10:00–18:00", dayoff: "Irregular holidays",
        price_range: "370¥ ~ 790¥", credit_card: "Yes",
        dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken ",
        peak_hours: "15:00~", avg_waiting_time: "None", reservation: "Not needed"
    },
	{
        name: "Kinosaki Ropeway Hiking", lat: 35.6227497, lng: 134.7972994, icon: "fa-mountain", color: "#795548",
        image: "https://i.imgur.com/jzRgYKt.jpeg", google_maps_link: "https://maps.app.goo.gl/GqEutHgPxvN5u2Z17",
        recommended_by: "Mr.Matsuo",
        highlight: "Hiking & Ropeway<br>This hiking trail allows you to fully experience Japan's rich nature firsthand while healthily refreshing your mind and body. Walking while enjoying the scenery provides a pleasant dose of exercise. Combining a relaxing time in the hot spring town with an active experience makes for an even more fulfilling stay. We also highly recommend taking a break at the summit cafe after your hike.",
        opening_hours: "09:10–17:10", dayoff: "Irregular holidays",
        price_range: "380¥ ~ 1.200¥", credit_card: "No",
        dine_in: "", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken ",
        peak_hours: "10:00–15:00", avg_waiting_time: "None", reservation: "Not needed"
    },
	{
        name: "Kinosaki Tokiwa Garden", lat: 35.6266333, lng: 134.8125176, icon: "fa-coffee", color: "#795548",
        image: "https://i.imgur.com/0QJOczz.jpeg", google_maps_link: "https://maps.app.goo.gl/EH6W9Fp6VhWTvkMT6",
        recommended_by: "Mr.Nakata (Manager)",
        highlight: "Coffee Milk<br>Housed in a gorgeous building with high ceilings and a modern-yet-traditional wooden exterior, this stylish spot serves incredible specialty coffee from DRIP & DROP COFFEE SUPPLY. Their absolute standout signature is the retro-cool Coffee Milk served in a nostalgic glass bottle—featuring a smooth, adult-friendly bittersweet flavor that is perfect to enjoy while sitting by the river or relaxing after a hot spring soak!",
        opening_hours: "08:00–18:00", dayoff: "Irregular holidays",
        price_range: "500¥ ~ 800¥", credit_card: "Yes",
        dine_in: "Dine-in / Takeout", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken",
        peak_hours: "10:00–15:00", avg_waiting_time: "5 min", reservation: "Not needed"
    }
		
        ],

        "Shopping & Souvenirs": [
            {
                name: "TAS", lat: 35.6252028, lng: 134.8124073, icon: "fa-shop", color: "#9C27B0",
                image: "https://i.imgur.com/cYZ6CDN.jpeg", google_maps_link: "https://maps.app.goo.gl/uRCWhHixCNAgi2fB7", recommended_by: "Mr.Kanyama", highlight: "Cookie. <br>This incredibly stylish, hidden gem of a cafe is famous for its warm, minimal aesthetic and exceptional bakes. Their Cookies are an absolute standout—perfectly thick, golden-baked, and loaded with premium ingredients. They manage to achieve that dream texture of being satisfyingly crisp around the edges while remaining soft and dense on the inside. Pair one with their expertly crafted specialty coffee for the perfect artistic afternoon break!",  opening_hours: "10:30–16:00", dayoff: "Sunday, Monday, Tuesday", price_range: "400¥ ~ 2.000¥", credit_card: "Yes", dine_in: "Dine in / Take out", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken", peak_hours: "None", avg_waiting_time: "None", reservation: "Not needed"
            },
            {
                name: "Chanoyu", lat: 35.6265861, lng: 134.8115356, icon: "fa-leaf", color: "#9C27B0",
                image: "https://i.imgur.com/XrClNSz.jpeg", google_maps_link: "https://maps.app.goo.gl/1BLWyexiuctM91GVA", recommended_by: "Mr.Yamamoto", highlight: "Baked Sweet Potato Flavored Green Tea .<br>This cozy, welcoming tea shop is famous for its passionate owner who loves sharing the rich world of Japanese tea. Their Baked Sweet Potato Flavored Green Tea is an absolute standout—combining a deeply comforting, fragrant tea base with the warm, naturally sweet, and comforting aroma of roasted sweet potatoes. It’s an incredibly relaxing and unique brew that pairs beautifully with their popular matcha roll cakes, making it the perfect healthy treat to warm up with while exploring the town!", opening_hours: "10:30–18:00", dayoff: "Wednesday", price_range: "500¥ ~ 1.620¥", credit_card: "Yes", dine_in: "Dine in / Take out", group_size: "Solo / couples / large groups", english_support: "English menu / English spoken", peak_hours: "None", avg_waiting_time: "None", reservation: "Not needed"
            }
			    ],

        "Pharmacy": [
            {
                name: "Oda Pharmacy", lat: 35.626636430598154, lng: 134.81161200256892, icon: "fa-shop", color: "#9C27B0", 
                image: "", google_maps_link: "https://maps.app.goo.gl/1Tz1j6cMyhnDERjt8", recommended_by: "Mr.Fukudame", highlight: "",  opening_hours: "09:00–17:00", dayoff: "Sunday", price_range: " ", credit_card: "Yes", dine_in: "No", group_size: " ", english_support: " ", peak_hours: "None", avg_waiting_time: "None", reservation: "Not needed"
            }
] // <-- THÊM DẤU NÀY
    };    // <-- THÊM DẤU NÀY ĐỂ ĐÓNG DATASET
    // =========================================================================
    // KHU VỰC ĐỘNG CƠ XỬ LÝ
    // =========================================================================
    const listContainer = document.getElementById('list-container');

    Object.keys(dataset).forEach(category => {
        const heading = document.createElement('div');
        heading.className = 'category-title';
        heading.innerText = category;
        listContainer.appendChild(heading);

        dataset[category].forEach(loc => {
            const htmlIcon = L.divIcon({
                html: `<div class="map-custom-marker" style="color: ${loc.color};"><i class="fa-solid ${loc.icon}"></i></div>`,
                className: 'custom-leaflet-div-icon',
                iconSize: [32, 32],
                iconAnchor: [16, 32],
                popupAnchor: [0, -32]
            });

            // Hiển thị Recommended và Highlight dạng HTML (hỗ trợ thẻ <br> xuống dòng)
            let popupContent = `
                <div class="popup-card">
                    ${loc.image ? `<img class="popup-img" src="${loc.image}" alt="${loc.name}">` : ''}
                    <div class="popup-title">🏪 ${loc.name}</div>
                    
                    ${loc.recommended_by ? `<div class="popup-meta"><i class="fa-solid fa-thumbs-up"></i> <b>Recommended by:</b> ${loc.recommended_by}</div>` : ''}
                    ${loc.highlight ? `<div class="popup-meta" style="color:#2b6cb0;"><i class="fa-solid fa-star"></i> <b>Highlight:</b> ${loc.highlight}</div>` : ''}
                    
                    <div class="popup-section-title">⏱️ Hours & Holidays</div>
                    <div class="popup-meta"><i class="fa-regular fa-clock"></i> <b>Opening Hours:</b> ${loc.opening_hours || '[Insert hours]'}</div>
                    <div class="popup-meta"><i class="fa-solid fa-calendar-xmark"></i> <b>Closed on:</b> ${loc.dayoff || '[Insert days off]'}</div>
            `;

            // Ẩn bảng thông tin hàng quán phức tạp đối với Suối nước nóng và Điểm xuất phát Ryokufu kaku
            if (category !== "Hot Springs (Sotoyu)" && category !== "Starting Point") {
                popupContent += `
                    <div class="popup-section-title">💰 Pricing & Payment</div>
                    <div class="popup-meta"><i class="fa-solid fa-money-bill-wave"></i> <b>Price Range:</b> ${loc.price_range || '[Insert budget]'}</div>
                    <div class="popup-meta"><i class="fa-regular fa-credit-card"></i> <b>Credit Card:</b> ${loc.credit_card || '[Yes / No / Cash Only]'}</div>
                    
                    <div class="popup-section-title">👥 Seating & Service</div>
                    <div class="popup-meta"><i class="fa-solid fa-chair"></i> <b>Dine-in:</b> ${loc.dine_in || '[Dine-in / Takeout only]'}</div>
                    <div class="popup-meta"><i class="fa-solid fa-users"></i> <b>Group Size:</b> ${loc.group_size || '[Best for solo / couples / large groups]'}</div>
                    <div class="popup-meta"><i class="fa-solid fa-language"></i> <b>English Support:</b> ${loc.english_support || '[English menu / spoken / None]'}</div>
                    
                    <div class="popup-section-title">⏳ Crowds & Bookings</div>
                    <div class="popup-meta"><i class="fa-solid fa-hourglass-half"></i> <b>Peak Hours:</b> ${loc.peak_hours || '[Insert busy times]'}</div>
                    <div class="popup-meta"><i class="fa-solid fa-stopwatch"></i> <b>Avg. Waiting Time:</b> ${loc.avg_waiting_time || '[Insert duration]'}</div>
                    <div class="popup-meta"><i class="fa-regular fa-calendar-check"></i> <b>Reservation:</b> ${loc.reservation || '[Not needed / Recommended / Required]'}</div>
                `;
            }

            if (loc.google_maps_link) {
                popupContent += `
                    <a href="${loc.google_maps_link}" target="_blank" class="gmaps-btn">
                        <i class="fa-solid fa-map-location-dot"></i> Open in Google Maps
                    </a>
                `;
            }

            popupContent += `</div>`;

            const marker = L.marker([loc.lat, loc.lng], { icon: htmlIcon }).addTo(map);
            marker.bindPopup(popupContent);

            const item = document.createElement('div');
            item.className = 'location-item';
            item.innerHTML = `
                <div class="location-item-icon" style="background-color: ${loc.color}20; color: ${loc.color};">
                    <i class="fa-solid ${loc.icon}"></i>
                </div>
                <div class="location-item-info">
                    <div class="location-item-name">${loc.name}</div>
                    <div class="location-item-desc">${loc.opening_hours ? `Open: ${loc.opening_hours}` : 'Click to see details'}</div>
                </div>
            `;

            item.addEventListener('click', () => {
                map.setView([loc.lat, loc.lng], 18, { animate: true, duration: 1 });
                marker.openPopup();
                if (window.innerWidth <= 768) {
                    document.getElementById('map').scrollIntoView({ behavior: 'smooth' });
                }
            });

            listContainer.appendChild(item);
        });
    });

</script>
</body>
</html>

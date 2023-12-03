ALTER SESSION SET NLS_DATE_FORMAT='DD-Mon-YYYY';

DROP TABLE outlets CASCADE CONSTRAINTS;
DROP TABLE staff CASCADE CONSTRAINTS; 
DROP TABLE menu CASCADE CONSTRAINTS; 
DROP TABLE orders CASCADE CONSTRAINTS;
DROP TABLE orderlines CASCADE CONSTRAINTS;
DROP TABLE customer CASCADE CONSTRAINTS;
DROP TABLE vehicles CASCADE CONSTRAINTS;
DROP TABLE feedback CASCADE CONSTRAINTS;
DROP TABLE suppliers CASCADE CONSTRAINTS;
DROP TABLE supplylines CASCADE CONSTRAINTS;
DROP TABLE ingredients CASCADE CONSTRAINTS;

DROP VIEW staff_customer_view;
DROP VIEW monthly_revenue_view;

CREATE TABLE outlets (
    outletID CHAR(5) PRIMARY KEY,
    telephone_num VARCHAR2(12) NOT NULL
                  CONSTRAINT chk_outlet_phone_num_digits
                  CHECK (REGEXP_LIKE(telephone_num, '^[0-9-]+$')),
    street VARCHAR(50) NOT NULL,
    suburb VARCHAR2(30) NOT NULL,
    postcode CHAR(5) NOT NULL,
    UNIQUE (telephone_num)
);

CREATE TABLE staff(
    staffID CHAR(7) PRIMARY KEY,
    outletID  CHAR(5) NOT NULL,
    title VARCHAR2(20) NOT NULL,
    first_name VARCHAR2(30)  NOT NULL,
    last_name VARCHAR2(30) NOT NULL,
    phone_num VARCHAR2(12) NOT NULL,
              CONSTRAINT chk_staff_phone_num_digits
              CHECK (REGEXP_LIKE(phone_num, '^[0-9-]+$')),
    street  VARCHAR2(100) NOT NULL,
    suburb VARCHAR2(100) NOT NULL,
    postcode NUMBER(5,0) NOT NULL,
    position VARCHAR2(40) NOT NULL,
    date_of_entry DATE NOT NULL,
    disciplinary_action VARCHAR2(400),
    FOREIGN KEY (outletID) REFERENCES outlets(outletID),
    UNIQUE(first_name, last_name,phone_num)
);

CREATE TABLE menu (
    itemID CHAR(6) PRIMARY KEY,
    description VARCHAR2(200) NOT NULL,
    food_type VARCHAR2(50) NOT NULL,
    price NUMBER(5,2) NOT NULL 
          CONSTRAINT chk_price
          CHECK (price > 0)
);

CREATE TABLE customer (
    customerID char(10) PRIMARY KEY,
    phone_num VARCHAR2(12) NOT NULL
              CONSTRAINT chk_phone_num_digits
              CHECK (REGEXP_LIKE(phone_num, '^[0-9-]+$')),
    title VARCHAR2(10) NOT NULL,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    street VARCHAR2(100),
    suburb VARCHAR2(50),
    postcode char(5),
    UNIQUE (phone_num)
);

CREATE TABLE orders (
    orderID char(12) PRIMARY KEY,
    customerID char(10) NOT NULL,
    order_date DATE NOT NULL,
    order_time TIMESTAMP(0) NOT NULL,
    order_method VARCHAR2(8) NOT NULL 
                 CONSTRAINT chk_order_method 
                 CHECK (order_method IN ('dine-in', 'delivery')),
    telephone_operator char(7),
    riderID char(7),
    outletID char(5) NOT NULL,
    UNIQUE (customerID, order_date, order_time),
    FOREIGN KEY (customerID) REFERENCES customer(customerID),
    FOREIGN KEY (telephone_operator) REFERENCES staff(staffID),
    FOREIGN KEY (riderID) REFERENCES staff(staffID),
    FOREIGN KEY (outletID) REFERENCES outlets(outletID)
);

CREATE TABLE orderlines (
    orderID CHAR(12),
    itemID CHAR(6),
    quantity NUMBER(2) NOT NULL
             CONSTRAINT chk_quantity
             CHECK (quantity > 0),
    PRIMARY KEY(orderID, itemID),
    FOREIGN KEY (orderID) REFERENCES orders(orderID),
    FOREIGN KEY (itemID) REFERENCES menu(itemID)
）;

CREATE TABLE vehicles (
    vehicleID VARCHAR2(20) PRIMARY KEY,
    brand VARCHAR2(30) NOT NULL,
    vehicle_type VARCHAR2(10) NOT NULL,
    date_of_purchase DATE NOT NULL,
    last_checked_date DATE NOT NULL, 
    mileage FLOAT(3) NOT NULL
            CONSTRAINT chk_mileage
            CHECK (mileage >=0),
    outletID CHAR(5),
    FOREIGN KEY (outletID) REFERENCES outlets(outletID),
    UNIQUE (vehicleID, outletID)
);

CREATE TABLE feedback (
    reviewID CHAR(10) PRIMARY KEY,
    customerID CHAR(10) NOT NULL,
    orderID CHAR(12) NOT NULL,
    review_score NUMBER(5) NOT NULL
                 CONSTRAINT chk_score
                 CHECK (review_score >= 1 AND review_score <= 5),
    review_text VARCHAR2(100),
    review_date DATE NOT NULL，
    FOREIGN KEY (customerID) REFERENCES customer(customerID),
    FOREIGN KEY (orderID) REFERENCES orders(orderID),
    UNIQUE (customerID, orderID)
);

CREATE TABLE ingredients (
    ingredientID CHAR(5) PRIMARY KEY,
    ingredient_type VARCHAR2(40) NOT NULL,
    quantity_in_stock NUMBER(5) NOT NULL,
    UNIQUE (ingredient_type)
);

CREATE TABLE suppliers (
    supplierID CHAR(5) PRIMARY KEY,
    ingredientID CHAR(5) NOT NULL,
    supplier_name VARCHAR(50) NOT NULL,
    contact_num VARCHAR(12) NOT NULL
                CONSTRAINT chk_cus_phone_num_digits
                CHECK (REGEXP_LIKE(contact_num, '^[0-9-]+$')),
    email VARCHAR(50) NOT NULL
          CONSTRAINT chk_email 
          CHECK (email LIKE '%_@__%.%_%'),
    FOREIGN KEY (ingredientID) REFERENCES ingredients(ingredientID)，
    UNIQUE (supplier_name),
    UNIQUE (email),
    UNIQUE (contact_num)
);

CREATE TABLE supplylines (
    outletID CHAR(5) NOT NULL,
    supplierID CHAR(5) NOT NULL,
    delivery_frequency VARCHAR2(20) NOT NULL,
                       CONSTRAINT chk_delivery_frequency
                       CHECK (delivery_frequency IN ('daily', 'weekly', 'monthly')),
    PRIMARY KEY (outletID, supplierID),
    FOREIGN KEY (outletID) REFERENCES outlets(outletID),
    FOREIGN KEY (supplierID) REFERENCES suppliers(supplierID)
);

INSERT INTO outlets VALUES ('OL001', '016-98765432', 'Jalan SS 21/1', 'Damansara Utama', '47400');
INSERT INTO outlets VALUES ('OL002', '011-65437890', 'Jalan Telawi 5', 'Bangsar Baru', '59100');
INSERT INTO outlets VALUES ('OL003', '012-4563742', 'Jalan SS 6/1', 'Kelana Jaya', '47301');
INSERT INTO outlets VALUES ('OL004', '03-51920607', 'Jalan Setia Prima E U13/E', 'Shah Alam', '47310');
INSERT INTO outlets VALUES ('OL005', '010-2345892', 'Jalan Setia Nusantara U13/N', 'Setia Alam', '40170');
INSERT INTO outlets VALUES ('OL006', '012-12345678', 'Jalan Kota Raja E 27/E', 'Klang', '41100');
INSERT INTO outlets VALUES ('OL007', '010-9876543', 'Jalan USJ 10/1G', 'Subang Jaya', '47600');

INSERT INTO staff VALUES ('SM12345', 'OL005', 'Ms.', 'Mei Ling', 'Low', '012-4567894', 'Jalan Apartment Puncak Baiduri', 'Cheras', 43200, 'Store Manager', '18-Jan-2019',NULL);
INSERT INTO staff VALUES ('DD09286', 'OL002', 'Mr.', 'Jaya', 'Changuuran', '013-5647283', 'Jalan Klang Lama', 'Old Klang', 58800, 'Delivery Driver', '09-Feb-2022', 'Suspended one week');
INSERT INTO staff VALUES ('SL23857', 'OL001', 'Mrs.', 'Shinna', 'Chopra', '012-5765784', 'Jalan Akik 7/3', 'Shah Alam', 40000, 'Shift Leader', '01-Apr-2018', NULL);
INSERT INTO staff VALUES ('SC27384', 'OL003', 'Mr.', 'Ali', 'Bin Muhammad Abu', '014-4859372', 'Jalan Alur 8/27', 'Shah Alam', 40000,'Store Cleaner', '29-Mar-2020', 'Fine');
INSERT INTO staff VALUES ('TM28475', 'OL005', 'Mr.', 'Jamal', 'Bin Abu Bakar', '016-4839528', 'Jalan SS 8/39', 'Kelana Jaya', 47300, 'Team Member', '30-May-2021', 'Suspended one week'); 
INSERT INTO staff VALUES ('TM36482', 'OL004', 'Ms.', 'Shelly', 'Lim', '019-2748576', 'Jalan Klang', 'Klang', 41100, 'Team Member','04-Jun-2017', NULL);
INSERT INTO staff VALUES ('SM36475', 'OL002', 'Dr.', 'Vesper', 'Noir', '017-2465738', 'Jalan BU6/12', 'Bandar Utama', 47800 ,'Store Manager','08-Jul-2019', NULL);  
INSERT INTO staff VALUES ('SL27384', 'OL003', 'Mrs.', 'Lei Chang', 'Tan', '014-3865759', 'Jalan Bangsa Baru', 'Bangsa Baru', 59100, 'Shift Leader', '04-Aug-2022',NULL); 
INSERT INTO staff VALUES ('SC37109', 'OL001', 'Mrs.', 'Kelly', 'Kang', '016-4487395', 'Jalan BU3/4', 'Bandar Utama', 47800, 'Store Cleaner', '08-Mar-2017', 'Fine');
INSERT INTO staff VALUES ('CC64270', 'OL005', 'Mr.', 'Eddy', 'Chan', '012-3456789', '30, Jalan Setia Indah U13/12q', 'Setia Alam', 40170, 'Counter Cashier', '12-Jan-2023', NULL);
INSERT INTO staff VALUES ('CC34058', 'OL004', 'Ms.', 'Nurul', 'Mustafa', '011-23456789', 'Jalan Setia Alam Street', 'Setia Alam', 40200, 'Counter Cashier', '28-Feb-2022', 'Suspended for one week');
INSERT INTO staff VALUES ('CC23948', 'OL005', 'Ms.', 'Siti', 'Jaffar', '019-87654321', 'Jalan Shah Alam Avenue', 'Shah Alam', 40450, 'Counter Cashier', '09-Mar-2022', NULL);
INSERT INTO staff VALUES ('CC92683', 'OL002', 'Mr.', 'Mohd', 'Azman', '013-65789012', '87, Jalan Terasek 8', 'Bangsar', 59100, 'Counter Cashier', '16-Apr-2022', NULL);
INSERT INTO staff VALUES ('CC19238', 'OL005', 'Ms.', 'Farah', 'Yusri', '018-43210987', 'Jalan Eco park', 'Setia Alam', 40200, 'Counter Cashier', '05-Jan-2022', NULL);
INSERT INTO staff VALUES ('CC05297', 'OL001', 'Mr.', 'Amir', 'Iskandar', '014-56789012', '17, Jalan SS 20/22', 'Damansara Kim', 47400, 'Counter Cashier', '21-Jun-2017', NULL);
INSERT INTO staff VALUES ('CC14384', 'OL007', 'Ms.', 'Xin Ling', 'Chia', '017-12345678', '17, Jalan USJ 3/4j, Usj 3', 'Subang Jaya', 47600, 'Counter Cashier', '14-Jul-2019', 'fine');
INSERT INTO staff VALUES ('CC02938', 'OL006', 'Ms.', 'Azizah', 'Hamid', '016-78901234', 'Jalan Bahagia', 'Klang', 41100, 'Counter Cashier', '30-Aug-2021', NULL);
INSERT INTO staff VALUES ('DD75103', 'OL005', 'Ms.', 'Nor', 'Ibrahim', '015-67890123', 'Jalan Seri Harmoni', 'Setia Alam', 40200, 'Delivery Driver', '11-Feb-2019', NULL);
INSERT INTO staff VALUES ('DD32104', 'OL004', 'Mr.', 'Hafiz', 'Abdul Razak', '012-78901234', 'Jalan Mutiara', 'Shah Alam', 40450, 'Delivery Driver', '24-Mar-2017', NULL);
INSERT INTO staff VALUES ('DD94275', 'OL005', 'Ms.', 'Zara', 'Ramlan', '012-34567890', 'Jalan Damai', 'Setia Alam', 40200, 'Delivery Driver', '27-Aug-2019', NULL);
INSERT INTO staff VALUES ('DD79462', 'OL001', 'Mr.', 'Amirul', 'Jamaluddin', '013-90123456', 'Jalan Damai Utama', 'Damansara Utama', 47800, 'Delivery Driver', '28-Feb-2018', 'Suspended for one week');
INSERT INTO staff VALUES ('DD12038', 'OL007', 'Mr.', 'Syed', 'Ismail Hashim', '019-23456789', 'Jalan Indah Makmur', 'Subang Jaya', 47600, 'Delivery Driver', '04-Apr-2017', NULL);
INSERT INTO staff VALUES ('DD29382', 'OL006', 'Ms.', 'Fatimah', 'Saidin', '019-78901234', 'Jalan Cendana Murni', 'Klang', 41100, 'Delivery Driver', '07-Apr-2021', 'Suspended for one week');
INSERT INTO staff VALUES ('SM12444', 'OL001', 'Ms.', 'Gigi', 'Tan', '012-4567894', '56, Jalan BU 2/11', 'Damansara Utama', 47900, 'Store Manager','30-Apr-2017', NULL);    
INSERT INTO staff VALUES ('SM12897', 'OL003', 'Mr.', 'Shawn', 'Li', '013-5647283', '68, Jalan Kelana Jaya 3/11','Kelana Jaya', 47301,'Store Manager', '15-Feb-2018', NULL);    
INSERT INTO staff VALUES ('SM87903', 'OL004', 'Ms.', 'Lily', 'Nim', '014-4859372', '57, Jalan Shah 9/10', 'Shah Alam', 40450, 'Store Manager', '11-Sep-2018', NULL);    
INSERT INTO staff VALUES ('SM12198', 'OL006', 'Mrs.', 'Mandy', 'Kok', '016-4839528', '69, Jalan Klang 5/67', 'Klang', 41100, 'Store Manager','18-May-2020', NULL);    
INSERT INTO staff VALUES ('SM38576', 'OL007', 'Ms.', 'Paula', 'Si', '019-2748576', '76, Jalan Subang Jaya 9/11', 'Subang Jaya', 47600, 'Store Manager', '13-Jun-2017', NULL);
INSERT INTO staff VALUES ('CC09374', 'OL004', 'Mr.', 'Faisal', 'Mahmod', '010-89012345', 'Jalan Murni', 'Shah Alam', 40450, 'Counter Cashier', '23-Jan-2023', 'fine');
INSERT INTO staff VALUES ('DD02938', 'OL003', 'Ms.', 'Aina', 'Abdullah Sani', '023-56789012', 'Jalan Mutiara Permai', 'Kelana Jaya', 47301, 'Delivery Driver', '11-Feb-2022', 'fine');
INSERT INTO staff VALUES ('DD83239', 'OL004', 'Mrs.', 'Kai Ming', 'Lim', '026-90123456', 'Jalan Mewah Indah', 'Shah Alam', 40200, 'Delivery Driver', '29-May-2023', NULL);
INSERT INTO staff VALUES ('CC29472', 'OL003', 'Ms.', 'Vasuda', 'Lal', '012-2394234', '36, Jalan SS 5d/8, Ss 5', 'Kelana Jaya', 47301, 'Counter Cashier', '12-Sep-2022', 'Verbal warning');
INSERT INTO staff VALUES ('CC12934', 'OL007', 'Mr.', 'Shan Yuan', 'Chew', '014-2394787', '45, Jalan USJ 11/4m, Usj 11', 'Subang Jaya', 47620, 'Counter Cashier', '26-Apr-2022', 'Verbal warning');

INSERT INTO menu VALUES ('PZ1001', 'Blazing Seafood with spicy sweet sour sauce, tuna, crabsticks, pineapples, capsicums, onions, mozzarella cheese.', 'Pizza', 30.00);
INSERT INTO menu VALUES ('PT2001', 'Spaghetti with Tomato Bolognaise sauce, topped with Chicken Meatballs.', 'Pasta', 20.00);
INSERT INTO menu VALUES ('SD3001', 'Huts poppers sausuage', 'Sides' , 5.50);
INSERT INTO menu VALUES ('DR4001', 'Red Bull', 'Drinks', 3.40);
INSERT INTO menu VALUES ('DS5001', 'Cheese cake with chocalte ice cream', 'Desserts', 8.90);
INSERT INTO menu VALUES ('DS6072', 'Brownie Ala Mode', 'Desserts', 9.40);
INSERT INTO menu VALUES ('DR3450', 'Coke','Drinks', 5.30);
INSERT INTO menu VALUES ('PZ2301', 'Island Suprime with thousand island sauce, crabsticks, tuna, pineapples, onions, mozzarella cheese.', 'Pizza', 28.00);
INSERT INTO menu VALUES ('PT1001', 'Spaghetti with Creamy Carbonara sauce, chicken rolls, mushrooms and herbs.', 'Pasta', 21.00);
INSERT INTO menu VALUES ('PT3101', 'Mushroom & Garlic Spaghetti', 'Pasta', 21.00);
INSERT INTO menu VALUES ('SD3102', 'Crazy Chicken Crunchies with bbq sauce', 'Sides', 25.00);
INSERT INTO menu VALUES ('PT4101', 'Penne pasta tossed in a savory marinara sauce with sautéed mushrooms.', 'Pasta', 19.90);
INSERT INTO menu VALUES ('PT3304', 'A crispy thin crust topped with BBQ chicken and caramelized onions.', 'Pizza', 23.60);
INSERT INTO menu VALUES ('DS2467', 'Rich chocolate lava cake with a molten center, served with a scoop of vanilla ice cream.','Desserts', 15.40);
INSERT INTO menu VALUES ('DR2365', '100 plus', 'Drinks', 5.50);
INSERT INTO menu VALUES ('DR4467', 'A refreshing blend of freshly squeezed oranges and pineapples.', 'Drinks',6.70);
INSERT INTO menu VALUES ('PZ4301', 'A classic Margherita pizza with fresh basil and tomatoes on a soft, chewy crust.', 'Pizza', 25.70);
INSERT INTO menu VALUES ('SD5164', 'Golden-brown onion rings with a zesty dipping sauce.', 'Sides', 6.60);
INSERT INTO menu VALUES（'DS9499', 'A delectable slice of creamy cheesecake topped with mixed berries.', 'Desserts', 10.10);
INSERT INTO menu VALUES ('PZ9067', 'A delightful vegetarian pizza with roasted bell peppers, black olives, and feta cheese.', 'Pizza', 20.90);

INSERT INTO customer VALUES ('C696284290', '012-3456789', 'Mr.', 'John', 'Doe', '17, Jalan USJ 3/4j, Usj 3', 'Subang Jaya', '47600');
INSERT INTO customer VALUES ('C984027513', '019-1284248', 'Mrs.', 'Xiu Ying', 'Lee', '22, Jalan Setia Damai U13/15f', 'Setia Alam', '40170');
INSERT INTO customer VALUES ('C784039281', '011-23909405', 'Mr.', 'Mohammed Ashrul', 'bin Hamizuddin', NULL, NULL, NULL);
INSERT INTO customer VALUES ('C532938109', '012-2866789', 'Mr.', 'John', 'Ngu', '65, Jalan Pualam 7/32, Seksyen 7', 'Shah Alam', '40000');
INSERT INTO customer VALUES ('C232934893', '012-5453196', 'Mrs.', 'Khairun Fairos', 'binti Yuszelan', '33, Lorong Sungai 8, Taman Kota Jaya', 'Klang', '41000');
INSERT INTO customer VALUES ('C239457108', '011-19718250', 'Mr.', 'David', 'Wong', '90, Jalan Terasek 1', 'Bangsar', '59100');
INSERT INTO customer VALUES ('C128302094', '017-6391219', 'Mrs.', 'Nithya', 'a/l Santhara', NULL, NULL, NULL);
INSERT INTO customer VALUES ('C930485752', '016-2136823', 'Mr.', 'Daniel', 'Tan', '20, Jalan Setia Indah U13/11n', 'Setia Alam', '40170');
INSERT INTO customer VALUES ('C124978532', '019-3020070', 'Mr.', 'Vijaya Krishnan', 'a/l Ravichandran', NULL, NULL, NULL);
INSERT INTO customer VALUES ('C123456789', '013-2446968', 'Mr.', 'Wei Chen', 'Lim', '37, JLN SS 21/24', 'Damansara Utama', '47400');
INSERT INTO customer VALUES ('C758203419', '011-51041592', 'Ms.', 'Nur Liyana Nashriq', 'binti Zulfadhli', NULL, NULL, NULL);
INSERT INTO customer VALUES ('C923649123', '011-11061791', 'Mrs.', 'Noor Raihah Hadzis', 'binti Sharum', '47, Jalan USJ 11/4l, Usj 11', 'Subang Jaya', '47620');
INSERT INTO customer VALUES ('C023850342', '014-3456789', 'Ms.', 'Xin Lin', 'Lan', NULL, NULL, NULL);
INSERT INTO customer VALUES ('C203770683', '017-8063737', 'Ms.', 'Priya', 'a/l Sothinathan', NULL, NULL, NULL);
INSERT INTO customer VALUES ('C370598463', '014-6199227', 'Mr.', 'Benjamin', 'Chou', '39, Jalan Tambatan 19/16, Seksyen 19', 'Shah Alam', '40300');
INSERT INTO customer VALUES ('C436285973', '013-2957384', 'Mr.', 'Alexander', 'Wong', 'Blok 4, Flat PKNS, Seksyen 16', 'Shah Alam', '40200');
INSERT INTO customer VALUES ('C239179283', '017-2047534', 'Ms.', 'Rashaa', 'binti Baheej', '31, Jalan USJ 13/1a, Usj 13', 'Subang Jaya', '47630');
INSERT INTO customer VALUES ('C348239940', '016-2394758', 'Mr.', 'Qaaid', 'bin Naaif', '15, Jalan Batu Mandi 26/29, Taman Bukit Saga', 'Shah Alam', '40400');
INSERT INTO customer VALUES ('C120398498', '012-2304859', 'Mr.', 'Kailash', 'Subramanian', '24, Jalan SS 21/52', 'Damansara Utama', '47400');
INSERT INTO customer VALUES ('C928390212', '013-2939348', 'Mrs.', 'Aarushi', 'Srivastava', '109, Jalan Terasek 7', 'Bangsar', '59100');

INSERT INTO orders VALUES ('DI2950478MQW', 'C696284290', '20-Mar-2023', TO_TIMESTAMP('17:23:10', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL007');
INSERT INTO orders VALUES ('DE9407536QDS', 'C984027513', '04-Feb-2023', TO_TIMESTAMP('19:21:57', 'HH24:MI:SS'), 'delivery', 'CC64270', 'DD75103', 'OL005');
INSERT INTO orders VALUES ('DI1583947JDG', 'C784039281', '01-Apr-2023', TO_TIMESTAMP('11:12:39', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL005');
INSERT INTO orders VALUES ('DE5682930CRW', 'C532938109', '19-Jun-2023', TO_TIMESTAMP('16:25:42', 'HH24:MI:SS'), 'delivery', 'CC34058', 'DD32104', 'OL004');
INSERT INTO orders VALUES ('DE7384956CDN', 'C984027513', '29-Mar-2023', TO_TIMESTAMP('12:31:59', 'HH24:MI:SS'), 'delivery', 'CC23948', 'DD94275', 'OL005');
INSERT INTO orders VALUES ('DI4761852OFD', 'C239457108', '23-Jul-2023', TO_TIMESTAMP('08:55:41', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL006');
INSERT INTO orders VALUES ('DE9023839NAS', 'C232934893', '18-Feb-2023', TO_TIMESTAMP('13:28:09', 'HH24:MI:SS'), 'delivery', 'CC92683', 'DD09286', 'OL002');
INSERT INTO orders VALUES ('DI1082389ASD', 'C128302094', '24-Jun-2023', TO_TIMESTAMP('16:03:12', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL002');
INSERT INTO orders VALUES ('DE2039485CAA', 'C930485752', '02-Feb-2023', TO_TIMESTAMP('11:39:55', 'HH24:MI:SS'), 'delivery', 'CC19238', 'DD75103', 'OL005');
INSERT INTO orders VALUES ('DI6389284ACE', 'C124978532', '12-Apr-2023', TO_TIMESTAMP('15:43:09', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL003');
INSERT INTO orders VALUES ('DE0123456ABC', 'C123456789', '02-Feb-2023', TO_TIMESTAMP('13:23:05', 'HH24:MI:SS'), 'delivery', 'CC05297', 'DD79462', 'OL001');
INSERT INTO orders VALUES ('DI1092837CEB', 'C758203419', '08-Apr-2023', TO_TIMESTAMP('18:22:44', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL003');
INSERT INTO orders VALUES ('DE0294982AOM', 'C923649123', '08-Apr-2023', TO_TIMESTAMP('08:17:21', 'HH24:MI:SS'), 'delivery', 'CC14384', 'DD12038', 'OL007');
INSERT INTO orders VALUES ('DI6543210DEF', 'C023850342', '10-Jul-2023', TO_TIMESTAMP('09:20:11', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL001');
INSERT INTO orders VALUES ('DI9028301HDS', 'C203770683', '06-Jul-2023', TO_TIMESTAMP('09:47:18', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL003');
INSERT INTO orders VALUES ('DE0128302ASW', 'C239457108', '09-Jul-2023', TO_TIMESTAMP('12:34:23', 'HH24:MI:SS'), 'delivery', 'CC02938', 'DD29382', 'OL006');
INSERT INTO orders VALUES ('DE2410956MDS', 'C370598463', '15-Mar-2023', TO_TIMESTAMP('14:06:32', 'HH24:MI:SS'), 'delivery', 'CC34058', 'DD32104', 'OL004');
INSERT INTO orders VALUES ('DI3439248SDJ', 'C436285973', '16-Jan-2023', TO_TIMESTAMP('18:23:04', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL004');
INSERT INTO orders VALUES ('DE1930248SVD', 'C124978532', '02=Jul-2023', TO_TIMESTAMP('10:23:22', 'HH24:MI:SS'), 'delivery', 'CC29472', 'DD02938', 'OL003');
INSERT INTO orders VALUES ('DE0239419EJD', 'C696284290', '12-May-2023', TO_TIMESTAMP('12:45:23', 'HH24:MI:SS'), 'delivery', 'CC12934', 'DD12038', 'OL007');
INSERT INTO orders VALUES ('DI9384274GHJ', 'C239179283', '12=Apr=2023', TO_TIMESTAMP('10:12:28', 'HH24:MI:SS'), 'delivery', 'CC64270', 'DD94275', 'OL007');
INSERT INTO orders VALUES ('DE2349238SEH', 'C348239940', '08-Jun-2023', TO_TIMESTAMP('12:29:34', 'HH24:MI:SS'), 'delivery', 'CC09374', 'DD83239', 'OL004');
INSERT INTO orders VALUES ('DI2934859ADE', 'C120398498', '30-May-2023', TO_TIMESTAMP('20:29:34', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL001');
INSERT INTO orders VALUES ('DI1239424SKE', 'C928390212', '12-Mar-2023', TO_TIMESTAMP('18:23:34', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL002');
INSERT INTO orders VALUES ('DI1293848AKI', 'C023850342', '12-Jul-2023', TO_TIMESTAMP('12:18:24', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL001');
INSERT INTO orders VALUES ('DI3829478MHK', 'C370598463', '20-May-2023', TO_TIMESTAMP('15:12:28', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL004');
INSERT INTO orders VALUES ('DI2946194UCS', 'C120398498', '17-Jun-2023', TO_TIMESTAMP('10:12:34', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL001');
INSERT INTO orders VALUES ('DI7482349BVM', 'C239457108', '20-Jul-2023', TO_TIMESTAMP('14:23:38', 'HH24:MI:SS'), 'dine-in', NULL, NULL, 'OL006');
INSERT INTO orders VALUES ('DE3849762KVN', 'C532938109', '28-Jul-2023', TO_TIMESTAMP('17:37:12', 'HH24:MI:SS'), 'delivery', 'CC09374', 'DD32104', 'OL004');
INSERT INTO orders VALUES ('DE1298352ZLK', 'C436285973', '30-Jul-2023', TO_TIMESTAMP('10:58:23', 'HH24:MI:SS'), 'delivery', 'CC34058', 'DD83239', 'OL004');

INSERT INTO orderlines VALUES ('DE9407536QDS', 'DR3450', 2);
INSERT INTO orderlines VALUES ('DE9407536QDS', 'SD3102', 3);
INSERT INTO orderlines VALUES ('DE9407536QDS', 'SD5164', 1);
INSERT INTO orderlines VALUES ('DE9407536QDS', 'PZ1001', 1);
INSERT INTO orderlines VALUES ('DE5682930CRW', 'DS9499', 1);
INSERT INTO orderlines VALUES ('DE5682930CRW', 'SD5164', 1);
INSERT INTO orderlines VALUES ('DE5682930CRW', 'SD3001', 1);
INSERT INTO orderlines VALUES ('DE5682930CRW', 'PT2001', 1);
INSERT INTO orderlines VALUES ('DE7384956CDN', 'DR2365', 4);
INSERT INTO orderlines VALUES ('DE7384956CDN', 'DS2467', 2);
INSERT INTO orderlines VALUES ('DE7384956CDN', 'SD3001', 3);
INSERT INTO orderlines VALUES ('DE7384956CDN', 'PT1001', 2);
INSERT INTO orderlines VALUES ('DE9023839NAS', 'PT2001', 2);
INSERT INTO orderlines VALUES ('DE9023839NAS', 'DR4467', 2);
INSERT INTO orderlines VALUES ('DE9023839NAS', 'DR3450', 2);
INSERT INTO orderlines VALUES ('DE9023839NAS', 'DR4001', 4);
INSERT INTO orderlines VALUES ('DE2039485CAA', 'DS5001', 2);
INSERT INTO orderlines VALUES ('DE0123456ABC', 'PZ2301', 3);
INSERT INTO orderlines VALUES ('DE0123456ABC', 'PT4101', 1);
INSERT INTO orderlines VALUES ('DE0123456ABC', 'PT3304', 1);
INSERT INTO orderlines VALUES ('DE0123456ABC', 'DS6072', 5);
INSERT INTO orderlines VALUES ('DE0294982AOM', 'PZ2301', 4);
INSERT INTO orderlines VALUES ('DE0294982AOM', 'PZ4301', 1);
INSERT INTO orderlines VALUES ('DE0294982AOM', 'DR3450', 2);
INSERT INTO orderlines VALUES ('DE0128302ASW', 'PZ9067', 2);
INSERT INTO orderlines VALUES ('DE0128302ASW', 'DR4001', 3);
INSERT INTO orderlines VALUES ('DE0128302ASW', 'PZ2301', 2);
INSERT INTO orderlines VALUES ('DE2410956MDS', 'PZ9067', 2);
INSERT INTO orderlines VALUES ('DE2410956MDS', 'DS9499', 3);
INSERT INTO orderlines VALUES ('DE2410956MDS', 'PT3101', 1);
INSERT INTO orderlines VALUES ('DE2410956MDS', 'SD5164', 2);
INSERT INTO orderlines VALUES ('DE2410956MDS', 'PT1001', 3);
INSERT INTO orderlines VALUES ('DI2950478MQW', 'DR3450', 1);
INSERT INTO orderlines VALUES ('DI2950478MQW', 'DR2365', 1);
INSERT INTO orderlines VALUES ('DI2950478MQW', 'DR4467', 2);
INSERT INTO orderlines VALUES ('DI2950478MQW', 'SD3001', 2);
INSERT INTO orderlines VALUES ('DI2950478MQW', 'PT1001', 2);
INSERT INTO orderlines VALUES ('DI1583947JDG', 'DR3450', 2);
INSERT INTO orderlines VALUES ('DI1583947JDG', 'PZ9067', 1);
INSERT INTO orderlines VALUES ('DI1583947JDG', 'PT3304', 1);
INSERT INTO orderlines VALUES ('DI1583947JDG', 'SD3001', 1);
INSERT INTO orderlines VALUES ('DI1583947JDG', 'DS5001', 1);
INSERT INTO orderlines VALUES ('DI1583947JDG', 'DS6072', 1);
INSERT INTO orderlines VALUES ('DI4761852OFD', 'PT2001', 1);
INSERT INTO orderlines VALUES ('DI1082389ASD', 'PZ1001', 1);
INSERT INTO orderlines VALUES ('DI1082389ASD', 'PT2001', 1);
INSERT INTO orderlines VALUES ('DI1082389ASD', 'DR3450', 2);
INSERT INTO orderlines VALUES ('DI6389284ACE', 'PZ2301', 4);
INSERT INTO orderlines VALUES ('DI6389284ACE', 'DS6072', 2);
INSERT INTO orderlines VALUES ('DI6389284ACE', 'SD5164', 1);
INSERT INTO orderlines VALUES ('DI6389284ACE', 'PT4101', 1);
INSERT INTO orderlines VALUES ('DI6389284ACE', 'DR3450', 6);
INSERT INTO orderlines VALUES ('DI1092837CEB', 'PT2001', 1);
INSERT INTO orderlines VALUES ('DI1092837CEB', 'PZ2301', 1);
INSERT INTO orderlines VALUES ('DI1092837CEB', 'SD3102', 1);
INSERT INTO orderlines VALUES ('DI1092837CEB', 'DR4001', 1);
INSERT INTO orderlines VALUES ('DI6543210DEF', 'PZ2301', 1);
INSERT INTO orderlines VALUES ('DI6543210DEF', 'DS2467', 2);
INSERT INTO orderlines VALUES ('DI6543210DEF', 'DR2365', 2);
INSERT INTO orderlines VALUES ('DI6543210DEF', 'PT2001', 3);
INSERT INTO orderlines VALUES ('DI6543210DEF', 'SD3001', 2);
INSERT INTO orderlines VALUES ('DI6543210DEF', 'DR4001', 1);
INSERT INTO orderlines VALUES ('DI9028301HDS', 'PT3101', 2);
INSERT INTO orderlines VALUES ('DI9028301HDS', 'PT4101', 1);
INSERT INTO orderlines VALUES ('DI9028301HDS', 'DR4467', 1);
INSERT INTO orderlines VALUES ('DI9028301HDS', 'SD5164', 1);
INSERT INTO orderlines VALUES ('DI9028301HDS', 'PT1001', 3);
INSERT INTO orderlines VALUES ('DI9028301HDS', 'DS6072', 2);
INSERT INTO orderlines VALUES ('DI3439248SDJ', 'DS5001', 1);
INSERT INTO orderlines VALUES ('DI3439248SDJ', 'PT2001', 2);
INSERT INTO orderlines VALUES ('DI3439248SDJ', 'PZ2301', 1);
INSERT INTO orderlines VALUES ('DI3439248SDJ', 'PT4101', 1);
INSERT INTO orderlines VALUES ('DI3439248SDJ', 'PT3304', 2);
INSERT INTO orderlines VALUES ('DI3439248SDJ', 'SD3001', 1);
INSERT INTO orderlines VALUES ('DE1930248SVD', 'PZ2301', 3);
INSERT INTO orderlines VALUES ('DE1930248SVD', 'PZ1001', 3);
INSERT INTO orderlines VALUES ('DE1930248SVD', 'DR4001', 4);
INSERT INTO orderlines VALUES ('DE1930248SVD', 'DR3450', 4);
INSERT INTO orderlines VALUES ('DE0239419EJD', 'DS6072', 2);
INSERT INTO orderlines VALUES ('DI9384274GHJ', 'DR3450', 2);
INSERT INTO orderlines VALUES ('DI9384274GHJ', 'DR4001', 1);
INSERT INTO orderlines VALUES ('DI9384274GHJ', 'DS2467', 1);
INSERT INTO orderlines VALUES ('DI9384274GHJ', 'PZ1001', 2);
INSERT INTO orderlines VALUES ('DI9384274GHJ', 'DS5001', 1);
INSERT INTO orderlines VALUES ('DE2349238SEH', 'DR4467', 6);
INSERT INTO orderlines VALUES ('DE2349238SEH', 'PZ4301', 4);
INSERT INTO orderlines VALUES ('DE2349238SEH', 'DS2467', 2);
INSERT INTO orderlines VALUES ('DE2349238SEH', 'DS9499', 1);
INSERT INTO orderlines VALUES ('DE2349238SEH', 'SD3001', 3);
INSERT INTO orderlines VALUES ('DE2349238SEH', 'DR3450', 1);
INSERT INTO orderlines VALUES ('DI2934859ADE', 'PT1001', 2);
INSERT INTO orderlines VALUES ('DI2934859ADE', 'DS5001', 2);
INSERT INTO orderlines VALUES ('DI2934859ADE', 'PT4101', 2);
INSERT INTO orderlines VALUES ('DI2934859ADE', 'PT3101', 2);
INSERT INTO orderlines VALUES ('DI2934859ADE', 'DR4001', 5);
INSERT INTO orderlines VALUES ('DI2934859ADE', 'PZ9067', 1);
INSERT INTO orderlines VALUES ('DI1239424SKE', 'SD3102', 1);
INSERT INTO orderlines VALUES ('DI1239424SKE', 'SD5164', 1);
INSERT INTO orderlines VALUES ('DI1239424SKE', 'SD3001', 1);
INSERT INTO orderlines VALUES ('DI1239424SKE', 'PT2001', 1);
INSERT INTO orderlines VALUES ('DI1293848AKI', 'PZ4301', 1);
INSERT INTO orderlines VALUES ('DI1293848AKI', 'DS5001', 1);
INSERT INTO orderlines VALUES ('DI3829478MHK', 'PT3101', 1);
INSERT INTO orderlines VALUES ('DI3829478MHK', 'SD3102', 2);
INSERT INTO orderlines VALUES ('DI2946194UCS', 'PZ2301', 1);
INSERT INTO orderlines VALUES ('DI2946194UCS', 'DR2365', 1);
INSERT INTO orderlines VALUES ('DI7482349BVM', 'PT3304', 2);
INSERT INTO orderlines VALUES ('DI7482349BVM', 'DS2467', 2);
INSERT INTO orderlines VALUES ('DI7482349BVM', 'SD5164', 1);
INSERT INTO orderlines VALUES ('DI7482349BVM', 'DR3450', 2);
INSERT INTO orderlines VALUES ('DE3849762KVN', 'SD3001', 2);
INSERT INTO orderlines VALUES ('DE3849762KVN', 'DS5001', 1);
INSERT INTO orderlines VALUES ('DE3849762KVN', 'PZ9067', 1);
INSERT INTO orderlines VALUES ('DE1298352ZLK', 'DS6072', 1);
INSERT INTO orderlines VALUES ('DE1298352ZLK', 'DR4467', 3);
INSERT INTO orderlines VALUES ('DE1298352ZLK', 'PT4101', 3);
INSERT INTO orderlines VALUES ('DE1298352ZLK', 'PZ4301', 2);

INSERT INTO vehicles VALUES ('ABC 1234', 'Honda', 'Motorbike', '21-Jan-2023', '29-Mar-2023', 4665,'OL005');
INSERT INTO vehicles VALUES ('DEF 456', 'Honda', 'Motorbike', '12-Feb-2021',  '02-Jul-2023', 52103,'OL002');
INSERT INTO vehicles VALUES ('HGI 5566', 'Yamaha', 'Motorbike', '27-Apr-2021','27-Apr-2023', 22210,'OL001');
INSERT INTO vehicles VALUES ('BNC 9997', 'Honda', 'Car','01-Feb-2023','30-Jun-2023', 4112,'OL003');
INSERT INTO vehicles VALUES ('PNP 2234', 'Yamaha', 'Motorbike', '28-Mar-2019', '02-Mar-2023',  200002,'OL005');
INSERT INTO vehicles VALUES ('WSP 9801', 'Yamaha', 'Motorbike', '01-Jun-2023',  '16-Jul-2023', 659,'OL004');
INSERT INTO vehicles VALUES ('SDE 2016', 'Yamaha', 'Motorbike', '04-Jan-2021',  '04-Jul-2023',  86123,'OL002');
INSERT INTO vehicles VALUES ('PTP 2012', 'Perodua', 'Car', '05-Jan-2022','05-Jun-2023',  10020,'OL003');
INSERT INTO vehicles VALUES ('BSS 4034', 'Yamaha', 'Motorbike', '26-Feb-2023','06-May-2023', 999,'OL001');
INSERT INTO vehicles VALUES ('WQS 151', 'Honda', 'Motorbike', '26-Aug-2021', '22-Jun-2023', 54560, 'OL004');
INSERT INTO vehicles VALUES ('WTY 8690', 'Yamaha', 'Motorbike', '22-Jan-2022', '30-Apr-2023', 25680, 'OL003');
INSERT INTO vehicles VALUES ('BPN 4468', 'Yamaha', 'Motorbike', '26-Dec-2021', '19-Mar-2023', 49800, 'OL007');
INSERT INTO vehicles VALUES ('SYE 6666', 'Perodua', 'Car', '01-Jan-2023', '26-Jul-2023', 6900, 'OL007');
INSERT INTO vehicles VALUES ('BNB 7779', 'Honda', 'Motorbike', '17-Feb-2022', '01-Dec-2022', 10972, 'OL004');
INSERT INTO vehicles VALUES ('JOR 7997', 'Yamaha', 'Motorbike', '04-Apr-2023', '06-Jul-2023', 579, 'OL001');
INSERT INTO vehicles VALUES ('JYP 3400', 'Honda', 'Car', '06-Sep-2019', '25-Dec-2022', 245601, 'OL002');
INSERT INTO vehicles VALUES ('SQW 5677', 'Yamaha', 'Motorbike', '09-May-2022', '30-May-2023', 12309, 'OL006');
INSERT INTO vehicles VALUES ('SMT 5900', 'Yamaha', 'Motorbike', '10-Nov-2021', '07-Feb-2023', 54061.32, 'OL006');
INSERT INTO vehicles VALUES ('PKL 9963', 'Yamaha', 'Motorbike', '29-Jul-2019', '05-Jul-2023', 167129, 'OL006');
INSERT INTO vehicles VALUES ('WJE 210', 'Honda', 'Car', '03-Mar-2019', '23-Mar-2023', 172110, 'OL006');

INSERT INTO feedback VALUES ('QEW2345678', 'C984027513', 'DE9407536QDS', 5, 'The pizza same as the picture', '04-Feb-2023');
INSERT INTO feedback VALUES ('AAA3246751', 'C532938109', 'DE5682930CRW', 5, 'Value for money', '20-Jun-2023');
INSERT INTO feedback VALUES ('BWS5900125', 'C984027513', 'DE7384956CDN', 4, 'Fast delivery', '29-Mar-2023');
INSERT INTO feedback VALUES ('PYS1994328', 'C232934893', 'DE9023839NAS', 3, 'Ordinary taste', '20-Feb-2023');
INSERT INTO feedback VALUES ('JCK1910909', 'C930485752', 'DE2039485CAA', 5, NULL, '03-Feb-2023');
INSERT INTO feedback VALUES ('BBM2067917', 'C123456789', 'DE0123456ABC', 4, 'The pizza same as the picture', '02-Feb-2023');
INSERT INTO feedback VALUES ('SWT2010710', 'C923649123', 'DE0294982AOM', 4, 'Delicious', '12-Apr-2023');
INSERT INTO feedback VALUES ('FFD5689012', 'C239457108', 'DE0128302ASW', 5, 'Fast delivery', '10-Feb-2023');
INSERT INTO feedback VALUES ('DFG2839407', 'C370598463', 'DE2410956MDS', 5, NULL, '15-Mar-2023');
INSERT INTO feedback VALUES ('ABB6703221', 'C124978532', 'DE1930248SVD', 3, 'It was not terrible, but it did not quite meet my expectations.', '02-Feb-2023');
INSERT INTO feedback VALUES ('BQA2690120', 'C696284290', 'DE0239419EJD', 5, 'The pizza same as the picture', '15-May-2023');
INSERT INTO feedback VALUES ('YGA2561122', 'C348239940', 'DE2349238SEH', 5, NULL, '8-Jun-2023');
INSERT INTO feedback VALUES ('UIO7781245', 'C120398498', 'DI2934859ADE', 5, 'Delicious', '30-May-2023');
INSERT INTO feedback VALUES ('MNL4421941', 'C928390212', 'DI1239424SKE', 5, NULL, '12-Mar-2023');
INSERT INTO feedback VALUES ('UPP8001760', 'C023850342', 'DI6543210DEF', 3, 'The food was average; some dishes were good, while others fell short in taste.', '10-Jun-2023');
INSERT INTO feedback VALUES ('RTZ6602934', 'C436285973', 'DE1298352ZLK', 2, 'The toppings are all over the place!', '30-Jul-2023');
INSERT INTO feedback VALUES ('TYW2910547', 'C436285973', 'DI3439248SDJ', 4, NULL, '16-Jan-2023');
INSERT INTO feedback VALUES ('QQI8678034', 'C239179283', 'DI9384274GHJ', 4, NULL, '12-Apr-2023');
INSERT INTO feedback VALUES ('KIM7943276', 'C696284290', 'DI2950478MQW', 5, 'Value for money', '20-Mar-2023');
INSERT INTO feedback VALUES ('KLM3659122', 'C784039281', 'DI1583947JDG', 5, NULL, '1-Apr-2023');

INSERT INTO ingredients VALUES ('FL100', 'Flour', 1000);
INSERT INTO ingredients VALUES ('SA230', 'Sauce', 1000);
INSERT INTO ingredients VALUES ('DY450', 'Dairy', 950);
INSERT INTO ingredients VALUES ('ME234', 'Meats', 500);
INSERT INTO ingredients VALUES ('VE257', 'Vegetable', 500);
INSERT INTO ingredients VALUES ('HE235', 'Herbs', 400);
INSERT INTO ingredients VALUES ('SP765', 'Spices', 300);
INSERT INTO ingredients VALUES ('SE657', 'Seafoods', 500);
INSERT INTO ingredients VALUES ('CD465', 'Carbonated Drinks',1000);
INSERT INTO ingredients VALUES ('CB836', 'Cake Base',3450);
INSERT INTO ingredients VALUES ('PN324', 'Pasta Noodles', 512);
INSERT INTO ingredients VALUES ('CO983', 'Coooking Oil', 742);
INSERT INTO ingredients VALUES ('ET293', 'Exotic Toppings', 264);
INSERT INTO ingredients VALUES ('CH346', 'Chocolate' , 456);
INSERT INTO ingredients VALUES ('FF930', 'Fresh Fruits', 180);
INSERT INTO ingredients VALUES ('IC937', 'Ice Cream', 200);
INSERT INTO ingredients VALUES ('JU127', 'Juice Drinks', 389);
INSERT INTO ingredients VALUES ('SD282', 'Stimulant Drinks', 500);
INSERT INTO ingredients VALUES ('NS039', 'Nuts and Seeds', 380);
INSERT INTO ingredients VALUES ('ST946', 'Sweetening Agent', 210);

INSERT INTO suppliers VALUES ('SP001', 'FL100', 'FlourCo Sdn Bhd', '012-6728875', 'info@flourco.com');
INSERT INTO suppliers VALUES ('SP002', 'SA230', 'SauceMasters Trading (M) Sdn Bhd', '018-9003728', 'sales@saucemasters.com');
INSERT INTO suppliers VALUES ('SP003', 'DY450', 'DairyDelights MY', '03-52738599', 'contact@dairydelights.com');
INSERT INTO suppliers VALUES ('SP004', 'ME234', 'MeatSupreme MY', '013-4675821', 'info@meatsupreme.com');
INSERT INTO suppliers VALUES ('SP005', 'VE257', 'VeggieFresh Supplies (M) Sdn Bhd', '03-62341234', 'sales@veggiefreshsupplies.com');
INSERT INTO suppliers VALUES ('SP006', 'SP765', 'SpiceEmporium (M) Sdn Bhd', '016-2550978', 'contact@spiceemporium.com');
INSERT INTO suppliers VALUES ('SP007', 'SE657', 'SeafoodHarvesters MY', '012-4567890', 'info@seafoodharvesters.com');
INSERT INTO suppliers VALUES ('SP008', 'CD465', 'BeverageKings Sdn Bhd', '011-9987334', 'sales@beveragekings.com');
INSERT INTO suppliers VALUES ('SP009', 'CB836', 'FlourMillers MY', '03-51234567', 'contact@flourmillers.com');
INSERT INTO suppliers VALUES ('SP010', 'HE235', 'GlobalSauces Trading (M) Sdn Bhd', '014-69337841', 'info@globalsauces.com');
INSERT INTO suppliers VALUES ('SP011', 'PN324', 'PastaNoodle World (M) Sdn Bhd', '03-51920607', 'info@pastanoodleworld.com');
INSERT INTO suppliers VALUES ('SP012', 'CO983', 'HealthyCooking Oils', '010-2345678', 'contact@healthycookingoils.com');
INSERT INTO suppliers VALUES ('SP013', 'ET293', 'ExoticPizzas MY', '012-43215678', 'sales@exoticpizzas.com');
INSERT INTO suppliers VALUES ('SP014', 'CH346', 'ChocoDelights Trading (M) Sdn Bhd', '016-98765432', 'info@chocodelights.com');
INSERT INTO suppliers VALUES ('SP015', 'FF930', 'FreshFruits MY', '011-65437890', 'contact@freshfruits.com');
INSERT INTO suppliers VALUES ('SP016', 'IC937', 'IceCream Paradise (M) Sdn Bhd', '014-92301673', 'sales@icecreamparadise.com');
INSERT INTO suppliers VALUES ('SP017', 'JU127', 'JuiceMania Enterprise', '019-67129837', 'info@juicemania.com');
INSERT INTO suppliers VALUES ('SP018', 'SD282', 'BeverageZone MY', '017-09129234', 'contact@beveragezone.com');
INSERT INTO suppliers VALUES ('SP019', 'NS039', 'NuttyNuts Trading (M) Sdn Bhd', '011-22438847', 'sales@nuttynuts.com');
INSERT INTO suppliers VALUES ('SP020', 'ST946', 'SweetTreats Sdn Bhd', '014-33188374', 'info@sweettreats.com');
INSERT INTO suppliers VALUES ('SP021', 'SA230', 'SauceFusion Enterprise', '03-51920551', 'saucefusion@email.com');
INSERT INTO suppliers VALUES ('SP022', 'DY450', 'CheezyMansion Sbn Bhd', '012-3953763', 'cheezy_mansion@gmail.com');
INSERT INTO suppliers VALUES ('SP023', 'ME234', 'MeatLover MY', '010-4026301', 'meat_lover@gmail.com');
INSERT INTO suppliers VALUES ('SP024', 'VE257', 'Up Grocer Sbn Bhd', '03-51930890', 'up_grocer@gmail.com');
INSERT INTO suppliers VALUES ('SP025', 'PN324', 'Italiano Enterprise', '03-51916783', 'italiano_enterprise@gmail.com');
INSERT INTO suppliers VALUES ('SP026', 'CD465', 'Bubbly Beverage Co.', '017-2384723', 'info@bubblybeverage.com');
INSERT INTO suppliers VALUES ('SP027', 'JU127', 'FreshSips Beverages', '014-9436274', 'contact@freshsips.com');
INSERT INTO suppliers VALUES ('SP028', 'SD282', 'EnergizeBrew Co.', '03-273482373', 'sales@energizebrewco.com');
INSERT INTO suppliers VALUES ('SP029', 'CD465', 'Fizztastic Beverages', '012-3456789', 'info@fizztasticbeverages.com');
INSERT INTO suppliers VALUES ('SP030', 'JU127', 'ZestyJuice Delights', '011-23456789', 'info@zestyjuicedelights.com');
INSERT INTO suppliers VALUES ('SP040', 'SD282', 'ReviveBrew Co.', '016-7890123', 'contact@revivebrewco.com');

INSERT INTO supplylines VALUES ('OL005', 'SP002', 'daily');
INSERT INTO supplylines VALUES ('OL002', 'SP002', 'daily');
INSERT INTO supplylines VALUES ('OL001', 'SP010', 'monthly');
INSERT INTO supplylines VALUES ('OL003', 'SP008', 'weekly');
INSERT INTO supplylines VALUES ('OL005', 'SP008', 'weekly');
INSERT INTO supplylines VALUES ('OL004', 'SP006', 'weekly');
INSERT INTO supplylines VALUES ('OL002', 'SP005', 'monthly');
INSERT INTO supplylines VALUES ('OL003', 'SP004', 'monthly');
INSERT INTO supplylines VALUES ('OL001', 'SP003', 'daily');
INSERT INTO supplylines VALUES ('OL006', 'SP001', 'weekly');
INSERT INTO supplylines VALUES ('OL007', 'SP009', 'daily');
INSERT INTO supplylines VALUES ('OL006', 'SP007', 'daily');
INSERT INTO supplylines VALUES ('OL001', 'SP011', 'weekly');
INSERT INTO supplylines VALUES ('OL001', 'SP012', 'monthly');
INSERT INTO supplylines VALUES ('OL002', 'SP013', 'weekly');
INSERT INTO supplylines VALUES ('OL007', 'SP014', 'weekly');
INSERT INTO supplylines VALUES ('OL004', 'SP015', 'daily');
INSERT INTO supplylines VALUES ('OL006', 'SP016', 'weekly');
INSERT INTO supplylines VALUES ('OL003', 'SP017', 'weekly');
INSERT INTO supplylines VALUES ('OL005', 'SP018', 'monthly');
INSERT INTO supplylines VALUES ('OL004', 'SP019', 'monthly');
INSERT INTO supplylines VALUES ('OL005', 'SP020', 'weekly');
INSERT INTO supplylines VALUES ('OL003', 'SP013', 'daily');
INSERT INTO supplylines VALUES ('OL001', 'SP021', 'weekly');
INSERT INTO supplylines VALUES ('OL003', 'SP022', 'daily');
INSERT INTO supplylines VALUES ('OL002', 'SP023', 'weekly');
INSERT INTO supplylines VALUES ('OL007', 'SP024', 'daily');
INSERT INTO supplylines VALUES ('OL005', 'SP025', 'weekly');
INSERT INTO supplylines VALUES ('OL007', 'SP020', 'monthly');
INSERT INTO supplylines VALUES ('OL007', 'SP026', 'weekly');
INSERT INTO supplylines VALUES ('OL007', 'SP027', 'weekly');
INSERT INTO supplylines VALUES ('OL003', 'SP028', 'weekly');
INSERT INTO supplylines VALUES ('OL002', 'SP029', 'weekly');
INSERT INTO supplylines VALUES ('OL002', 'SP030', 'weekly');
INSERT INTO supplylines VALUES ('OL007', 'SP040', 'weekly');

/*As part of our customer service tradition to provide personalized monthly order summaries to valued customers, 
we have decided to create an index on the order_date column. This decision is based on the frequent retrieval of 
order information for this purpose. With the index in place, the database engine can quickly locate the relevant orders, 
making the query execution faster and more efficient.*/
CREATE INDEX idx_order_date ON orders (order_date);

/*This index improves the database's ability to swiftly locate and access relevant data from the orderlines table during outlet 
sales analysis. This optimization is essential because sales and outlet performance analysis is performed on a monthly basis and 
involves aggregating and processing large volumes of data. */
CREATE INDEX idx_orderlines_revenue ON orderlines (orderID, itemID, quantity);

/*provides information about staff members who have interacted with customers through orders*/
CREATE VIEW staff_customer_view AS
SELECT sf.staffID, sf.first_name AS staff_first_name, sf.last_name AS staff_last_name, c.customerID, c.street AS customer_street, c.suburb AS customer_suburb
FROM staff sf, orders odr, customer c
WHERE (sf.staffID = odr.telephone_operator OR sf.staffID = odr.riderID)
AND odr.customerID = c.customerID;

/*provides a summary of monthly revenue for each outlet based on the last month of order data*/
CREATE VIEW monthly_revenue_view AS
SELECT ol.outletID, TO_CHAR(odr.order_date, 'Mon-YYYY') AS month, SUM(odrln.quantity * m.price) AS total_revenue
FROM outlets ol, orders odr, orderlines odrln, menu m
WHERE ol.outletID = odr.outletID
AND odr.orderID = odrln.orderID
AND odrln.itemID = m.itemID
AND oDR.order_date >= TRUNC(ADD_MONTHS(SYSDATE, -1), 'MONTH') -- Filter orders for the last month
AND oDR.order_date < TRUNC(SYSDATE, 'MONTH') -- Exclude any orders from the current month
GROUP BY ol.outletID, TO_CHAR(odr.order_date, 'Mon-YYYY');


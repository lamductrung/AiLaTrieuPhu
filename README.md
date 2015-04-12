## Game Ai La Trieu Phu
    * Xay dung game Ai La Trieu Phu
      - Game Ai La Trieu Phu duoc xay dung voi 15 cau hoi voi 15 muc do kho tu thap den cao
	  - Moi cau hoi chi co mot dap an dung 
	  - Nguoi choi duoc su dung 3 su tru giup sau
		+ Tro giup tu khan gia
		+ Goi dien cho nguoi than
		+ Lua chon 50/50
	*1 tao bang questions.
		CREATE TABLE questions(
			id SERIAL NOT NULL ,
			name CHARACTER VARYING(150) NOT NULL ,
			A CHARACTER VARYING(50) NOT NULL ,
			B CHARACTER VARYING(50) NOT NULL ,
			C CHARACTER VARYING(50) NOT NULL ,
			D CHARACTER VARYING(50) NOT NULL ,
			CONSTRAINT questions_pkey PRIMARY KEY (id),
			CONSTRAINT questions_name_key UNIQUE (name)
			)

	1.1 tao 100 cau hoi
			CREATE OR REPLACE FUNCTION randoms(numeric, numeric)
			RETURNS numeric AS
			$$
			SELECT ($1 + ($2 - $1) * random())::numeric;
			$$ LANGUAGE 'sql' VOLATILE;
			-- SELECT addQuestion();
		CREATE OR REPLACE FUNCTION addQuestion()
		RETURNS void AS
		$BODY$
			DECLARE
			x INT;
			y INT;
			BEGIN 
			FOR i IN 1..40
			LOOP 
			x = randoms(1,100) :: INTEGER;
			y = randoms(1,100):: INTEGER;
			INSERT INTO questions (name,A,B,C,D) VALUES (concat(x , ' + ' , y , ' = ?'),x + y, x - y, x * y, x / y);
			END LOOP;
			FOR i IN 1..30
			LOOP
			x = randoms(100,200) :: INTEGER;
			y = randoms(100,200):: INTEGER;
			INSERT INTO questions (name,A,B,C,D) VALUES (concat(x , ' + ' , y , ' = ?'),x + y, x - y, x * y, x / y);
			END LOOP ;
			FOR i IN 1..30
			LOOP
			x = randoms(500,1000) :: INTEGER;
			y = randoms(500,1000):: INTEGER;
			INSERT INTO questions (name,A,B,C,D) VALUES (concat(x , ' + ' , y , ' = ?'),x + y, x - y, x * y, x / y);
			END LOOP ;
			END
		$BODY$
		LANGUAGE plpgsql;

		SELECT addQuestion();

	1.2 tim
		SELECT name,A,B,C,D,A AS DapAnDung
		from questions
		WHERE id = 3

	2.1 them truong do kho
		ALTER TABLE questions ADD COLUMN difficult INTEGER;
            CREATE OR REPLACE FUNCTION addDiff()
              RETURNS VOID AS
		$$
		DECLARE
		r RECORD;
		BEGIN
		FOR r in
		SELECT id,A FROM questions
		LOOP
		IF to_number(r.a,'9999') < 200
		THEN UPDATE questions set difficult = 1 where id = r.id ;
          ELSEIF to_number(r.a,'9999') < 400
            THEN UPDATE questions set difficult = 2 where id = r.id ;
          ELSE UPDATE questions set difficult = 3 where id = r.id ;
        END IF ;
		END LOOP;
		END 
      
		$$
		LANGUAGE plpgsql;
		SELECT addDiff();
		ALTER TABLE questions ALTER COLUMN difficult SET NOT NULL;

	2.2 truy van cau hoi theo muc do kho
		SELECT difficult as mucdo, count(id) AS soluong FROM questions
		GROUP BY mucdo

	2.3 truy van ngau nhien theo muc do
		SELECT * FROM questions WHERE difficult = 1 ORDER BY random() LIMIT 1;
	2.4 lay ra 15 cau hoi
		CREATE OR REPLACE FUNCTION getQuestion()
		RETURNS TABLE(
			id integer, 
			name character varying(150),
			a CHARACTER VARYING(50),
			b CHARACTER VARYING(50),
			c CHARACTER VARYING(50),
			d CHARACTER VARYING(50),
			difficult INTEGER
		)
		AS $$
		DECLARE
		BEGIN
		RETURN QUERY
			(
			SELECT *
			FROM questions
			WHERE questions.difficult = 1
			ORDER BY random()
			LIMIT 5
			)
		UNION ALL
			(
			SELECT *
			FROM questions
			WHERE questions.difficult = 2
			ORDER BY random()
			LIMIT 5
			)
			UNION ALL
			(
			SELECT *
			FROM questions
			WHERE questions.difficult = 3
			ORDER BY random()
			LIMIT 5
			);

		END;
		$$
		LANGUAGE plpgsql;
		SELECT getQuestion();
	3
		CREATE TABLE config(
			id SERIAL NOT NULL ,
			price INTEGER NOT NULL ,
			CONSTRAINT config_pkey PRIMARY KEY (id)
			);
		INSERT INTO config(price) VALUES (200000);
		INSERT INTO config(price) VALUES (400000);
		INSERT INTO config(price) VALUES (600000);
		INSERT INTO config(price) VALUES (1000000);
		INSERT INTO config(price) VALUES (2000000);
		INSERT INTO config(price) VALUES (3000000);
		INSERT INTO config(price) VALUES (6000000);
		INSERT INTO config(price) VALUES (10000000);
		INSERT INTO config(price) VALUES (14000000);
		INSERT INTO config(price) VALUES (22000000);
		INSERT INTO config(price) VALUES (30000000);
		INSERT INTO config(price) VALUES (40000000);
		INSERT INTO config(price) VALUES (60000000);
		INSERT INTO config(price) VALUES (85000000);
		INSERT INTO config(price) VALUES (150000000);

	4.1 lay 2 trong 4 cau tra loi gom 1 cau dung
		CREATE OR REPLACE FUNCTION bo_di_2_phuong_an_sai(id INTEGER)
			RETURNS TABLE(
			name CHARACTER VARYING(150),
			t CHARACTER VARYING(50),
			f CHARACTER VARYING(50)
			) AS $$    
		DECLARE 
		rand INTEGER;
		BEGIN
		rand = round(random() * 10);
		rand = rand % 3;
		CASE 
		WHEN rand = 0 THEN RETURN QUERY SELECT questions.name,A,B FROM questions WHERE questions.id = $1;
		WHEN rand = 1 THEN RETURN QUERY SELECT questions.name,A,C FROM questions WHERE questions.id = $1;
		ELSE RETURN QUERY SELECT questions.name,A,D FROM questions WHERE questions.id = $1;
		END CASE;
		END 
		$$
		LANGUAGE plpgsql;

		SELECT bo_di_2_phuong_an_sai(1);

	4.2 lay ngau nhien ti le khan gia
		CREATE OR REPLACE FUNCTION hoi_y_kien_khan_gia()
			RETURNS TABLE(
			A INTEGER,
			B INTEGER,
			C INTEGER,
			D INTEGER
		) AS $$
		DECLARE 
			d INTEGER;
			a INTEGER;
			b INTEGER;
			c INTEGER;
		BEGIN
			d = 100;
			a = round(randoms(0,d));
			d = d - a;
			b = round(randoms(0,d));
			d = d - b;
			c = round(randoms(0,d));
			d = d - c;
		RETURN QUERY
			SELECT A,B,C,D;
		END
		$$
		LANGUAGE plpgsql;
		SELECT hoi_y_kien_khan_gia();

	4.3 ngau nhien dap an cua 1 cau
		CREATE OR REPLACE FUNCTION goi_dien_thoai_cho_nguoi_than(id INTEGER)
		returns TABLE(res CHARACTER VARYING(50)) AS $$
		DECLARE 
		rand INTEGER;
		BEGIN 
			rand = round(randoms(1,4));
			CASE
				WHEN rand = 1 THEN RETURN QUERY SELECT questions.a FROM questions WHERE questions.id = $1;
				WHEN rand = 2 THEN RETURN QUERY SELECT questions.b FROM questions WHERE questions.id = $1;
				WHEN rand = 3 THEN RETURN QUERY SELECT questions.c FROM questions WHERE questions.id = $1;
				ELSE RETURN QUERY SELECT questions.d FROM questions WHERE questions.id = $1;
			END CASE ;
		END 
		$$
		LANGUAGE plpgsql;

		SELECT goi_dien_thoai_cho_nguoi_than(1);

	5 tao bang nguoi choi
		CREATE TABLE users(
			id SERIAL NOT NULL ,
			name CHARACTER VARYING(50) NOT NULL ,
			starttime TIMESTAMP NOT NULL ,
			money INTEGER NOT NULL,
			CONSTRAINT users_pkey PRIMARY KEY (id)
		)

	5.1 tao du lieu 100 nguoi choi
		CREATE OR REPLACE FUNCTION adduser()
			RETURNS void AS $$
			DECLARE 
			ho CHARACTER VARYING(17);
			dem CHARACTER VARYING(17);
			ten CHARACTER VARYING(17);
			rand INTEGER;
		BEGIN 
		FOR i IN 1..5
			LOOP 
				CASE 
				WHEN i = 1 THEN ho = 'Nguyen';
				WHEN i = 2 THEN ho = 'Le';
				WHEN i = 3 THEN ho = 'Dang';
				WHEN i = 4 THEN ho = 'Dinh';
				ELSE ho = 'Tran';
				END CASE ;
		FOR j IN 1..5
			LOOP
				CASE
				WHEN j = 1 THEN dem = 'Duc';
				WHEN j = 2 THEN dem = 'Van';
				WHEN j = 3 THEN dem = 'Thi';
				WHEN j = 4 THEN dem = 'Cong';
				ELSE dem = 'Hoang';
				END CASE ;
		FOR k IN 1..4
            LOOP
				CASE
				WHEN k = 1 THEN ten = 'Trung';
				WHEN k = 2 THEN ten = 'Vu';
				WHEN k = 3 THEN ten = 'Vinh';
				ELSE ten = 'Nam';
				END CASE ;
				rand = round(randoms(1,15));
				INSERT INTO users(name,starttime,money) VALUES (concat(ho,' ',dem,' ',ten),now(),(SELECT price FROM config WHERE id = rand));
				END LOOP ;
			END LOOP ;
		END LOOP ;
		END 
      
		$$
		LANGUAGE plpgsql;

		SELECT adduser();

	5.2 select 10 nguoi diem cao nhat tu cao xuong thap
		SELECT * FROM users ORDER BY money DESC LIMIT 10;

	6.1 tim kiem cau hoi
		CREATE OR REPLACE FUNCTION searchquestion(n CHARACTER VARYING(50))
			RETURNS TABLE(
			id INTEGER,
			name CHARACTER VARYING(150),
			a CHARACTER VARYING(50),
			b CHARACTER VARYING(50),
			c CHARACTER VARYING(50),
			d CHARACTER VARYING(50)
			) AS $$
				DECLARE
			ch CHARACTER VARYING(50);
			BEGIN
			ch = concat('%',$1,'%');
			return QUERY 
			SELECT questions.id,questions.name,questions.a,questions.b,questions.c,questions.d
			FROM questions 
			WHERE questions.id LIKE ch
               OR questions.name LIKE ch
               OR questions.a LIKE ch
               OR questions.b LIKE ch
               OR questions.c LIKE ch
               OR questions.d LIKE ch;
		END 

		$$
		LANGUAGE plpgsql;

		SELECT searchquestion('2');

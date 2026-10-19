# საიტის გაშვება სერვერზე (Windows და Mac) — ნაბიჯ-ნაბიჯ
# Launching the site on a server — step by step

ეს ინსტრუქცია ერთნაირად მუშაობს ნებისმიერ Linux სერვერზე (Hetzner, DigitalOcean და ა.შ.).
ქვემოთ ყველა ბრძანება, რომელიც "თქვენს კომპიუტერზე" წერია, სრულდება:
- **Mac:** პროგრამა *Terminal* (Spotlight-ში ჩაწერეთ "Terminal")
- **Windows 10/11:** პროგრამა *PowerShell* (Start-ში ჩაწერეთ "PowerShell")

ბრძანებები ორივეგან ერთნაირია, სადაც განსხვავებაა, მითითებულია.

---

## 0. რა დაგჭირდებათ
1. სერვერი (VPS): Ubuntu 24.04, მინიმუმ 2 GB მეხსიერება.
2. დომენი და წვდომა მის DNS ჩანაწერებზე.
3. ფაილი `profesia-backend.zip`.
4. Verify.ge-ს მისამართი და გასაღები (SMS-ისთვის).

## 1. SSH გასაღები (ერთხელ, თქვენს კომპიუტერზე)
SSH გასაღები სერვერზე პაროლის გარეშე, უსაფრთხო შესვლაა.

```
ssh-keygen -t ed25519
```
ყველა კითხვაზე დააჭირეთ Enter (ან დაწერეთ პაროლი გასაღებისთვის, ეს უფრო უსაფრთხოა).
შემდეგ აჩვენეთ **საჯარო** გასაღები და დააკოპირეთ:

- Mac: `cat ~/.ssh/id_ed25519.pub`
- Windows (PowerShell): `type $env:USERPROFILE\.ssh\id_ed25519.pub`

გასაღები იწყება `ssh-ed25519`-ით. **არასოდეს გაუგზავნოთ ფაილი `id_ed25519` (.pub-ის გარეშე) არავის**, ის პირადია.

## 2. სერვერის შექმნა
პროვაიდერის საიტზე (მაგ. Hetzner Cloud): ახალი სერვერი → სურათი **Ubuntu 24.04** → პატარა ტიპი (მაგ. CX23) → SSH key-ში ჩასვით წინა ნაბიჯის გასაღები → შექმნა.
შექმნის შემდეგ ნახავთ **IP მისამართს** (მაგ. 203.0.113.10). ჩაიწერეთ.

რეკომენდებულია ჩართოთ პროვაიდერის **ავტომატური ბექაპი** (Hetzner-ზე "Backups").

## 3. სერვერზე შესვლა
```
ssh root@თქვენი-IP
```
პირველად ჰკითხავს "yes/no", დაწერეთ `yes`. ახლა სერვერზე ხართ.

## 4. სერვერის მომზადება (სერვერზე)
```
apt update && apt -y upgrade
apt -y install unzip ufw
curl -fsSL https://get.docker.com | sh
ufw allow OpenSSH
ufw allow 80
ufw allow 443
ufw --force enable
```
ბოლო სამი ბრძანება firewall-ს რთავს: გარედან მხოლოდ SSH, HTTP და HTTPS ჩანს.

## 5. ფაილების ატვირთვა
**ახალი ფანჯარა გახსენით თქვენს კომპიუტერზე** (სერვერიდან გამოსვლა არ არის საჭირო), გადადით საქაღალდეში, სადაც zip ფაილია, და გაუშვით:
```
scp profesia-backend.zip root@თქვენი-IP:/root/
```
შემდეგ სერვერზე:
```
cd /root
unzip -o profesia-backend.zip
cd profesia-backend
```

## 6. პარამეტრები (.env)
```
cp .env.example .env
chmod 600 .env
nano .env
```
შეავსეთ (მნიშვნელობები ტყუპი ბრჭყალების გარეშე):
- `DOMAIN=profesia.online.ge` (თქვენი დომენი)
- `APP_SECRET=` — შემთხვევითი გრძელი ტექსტი. მის მისაღებად სერვერზე: `openssl rand -hex 32`
- `ADMIN_KEY=` — სხვა შემთხვევითი ტექსტი (`openssl rand -hex 32`)
- `SMS_PROVIDER=verifyge`, `VERIFYGE_URL=...`, `VERIFYGE_API_KEY=...` (Verify.ge-ს კაბინეტიდან)

`nano`-ში შენახვა: **Ctrl+O**, Enter, გასვლა: **Ctrl+X**.

> ⚠️ `APP_SECRET` და `ADMIN_KEY` შეინახეთ უსაფრთხო ადგილას (მაგ. პაროლების მენეჯერში). `APP_SECRET`-ის დაკარგვის ან შეცვლის შემთხვევაში ყველა მომხმარებელი გამოვა სისტემიდან.

## 7. DNS (დომენი → სერვერი)
დომენის მიმწოდებლის პანელში დაამატეთ **A ჩანაწერი**: სახელი — თქვენი დომენი (profesia.online.ge), მნიშვნელობა — სერვერის IP.
10-30 წუთში (ხანდახან მეტში) დაიწყებს მუშაობას. შემოწმება თქვენს კომპიუტერზე: `ping profesia.online.ge` — უნდა გამოჩნდეს თქვენი IP.

## 8. გაშვება (სერვერზე, საქაღალდეში profesia-backend)
```
docker compose up -d --build
```
პირველად რამდენიმე წუთი სჭირდება. შემდეგ:
```
docker compose ps            # ორივე სერვისი უნდა იყოს "running"
docker compose logs -f app   # ლოგები (გამოსვლა: Ctrl+C)
```
ბრაუზერში გახსენით `https://თქვენი-დომენი`. HTTPS სერტიფიკატს Caddy თვითონ იღებს (მხოლოდ მაშინ, როცა DNS უკვე სწორია).

## 9. შემოწმება
- `https://თქვენი-დომენი/healthz` უნდა აჩვენოს `{"ok":true}`.
- დარეგისტრირდით ახალი ნომრით. Verify.ge-ს საცდელი ნომერია +995568000865, კოდი 123456.
- ადმინ-გვერდი: `https://თქვენი-დომენი/admin.html` (გასაღები = `ADMIN_KEY`).

## 10. ბექაპი (ძალიან მნიშვნელოვანია)
ბაზა და ატვირთული ფოტო/ვიდეოები ცხოვრობს Docker-ის volume-ში `profesia_data`.
ასლის გაკეთება (სერვერზე):
```
docker compose exec app node backup.js
```
ასლი ინახება volume-შიც (`/data/backups`), მაგრამ ეს **არ არის სრული დაცვა**, რადგან იმავე სერვერზეა. გადმოწერეთ თქვენს კომპიუტერზე:
```
docker compose cp app:/data/backups ./backups
```
და თქვენს კომპიუტერზე: `scp -r root@თქვენი-IP:/root/profesia-backend/backups .`

ავტომატურად ყოველ ღამე (სერვერზე `crontab -e`, ბოლოში დაამატეთ):
```
0 4 * * * cd /root/profesia-backend && docker compose exec -T app node backup.js
```
> ⚠️ **არასოდეს გაუშვათ** `docker compose down -v`: ეს წაშლის ბაზასა და ყველა ფაილს. ჩვეულებრივი `docker compose down` უსაფრთხოა.

## 11. განახლება (ახალი ვერსიის დადება)
1. ახალი zip ატვირთეთ (ნაბიჯი 5).
2. სერვერზე: `cd /root && unzip -o profesia-backend.zip && cd profesia-backend && docker compose up -d --build`
   `.env` ფაილი არ იშლება, ბაზა და ფაილები რჩება.

## 12. თუ რამე არ მუშაობს
- საიტი არ იხსნება: `docker compose ps` და `docker compose logs --tail 100 caddy app`
- HTTPS ვერ ჩაირთო: ჩვეულებრივ DNS ჯერ არ არის სწორი (ნაბიჯი 7).
- შესვლა გამოდის HTTP-ზე: HTTPS-ზე გადადით (cookie-ები მხოლოდ HTTPS-ზე მუშაობს).
- დროებითი ტესტი მხოლოდ IP-ით (დომენის გარეშე): `.env`-ში `DOMAIN=:80` და `COOKIE_SECURE=0`, მერე `docker compose up -d`. **ტესტის შემდეგ ეს ორი სტრიქონი აუცილებლად დააბრუნეთ.**

// Veri Yapıları (örnek)
// SystemState: {active: bool, alarm_active: bool, alarm_level: int, last_event: Event}
// Event: {type, timestamp, details}
// Sensors: {motion_sensors[], door_window_sensors[], cameras[]}
// owner_home(): bool  -> ev sahibi mi kontrolü (geofencing / manuel durum)

FUNCTION main():
    system <- load_system_state()
    WHILE TRUE:                        // Ana DÖNGÜ (Sürekli)
        IF NOT system.active:
            sleep(IDLE_INTERVAL)       // sistem pasif ise kısa uyku
            CONTINUE

        // 1) Sensörleri oku
        events <- []
        FOR each m IN motion_sensors:
            IF m.detected():
                events.append(Event("motion", now(), m.id))
        FOR each d IN door_window_sensors:
            IF d.opened():
                events.append(Event("door_window", now(), d.id))
        FOR each c IN cameras:
            // kameralar sadece tetiklenirse kaydetilecek (aktif tetiklemelerde)
            IF c.tamper_detected():
                events.append(Event("camera_tamper", now(), c.id))

        // 2) Tehdit değerlendirme - seviye belirleme
        level <- 0
        IF exists event in events of type "door_window":
            level <- max(level, 3)            // kapı/pencere açılması => yüksek
        IF exists event in events of type "motion":
            level <- max(level, 2)            // hareket => orta
        IF multiple motion sensors triggered recently OR camera_tamper:
            level <- max(level, 3)            // yoğun aktivite => yüksek
        IF events is empt

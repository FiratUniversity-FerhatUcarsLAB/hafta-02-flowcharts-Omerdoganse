// Üniversite Ders Kayıt Sistemi - PSEUDOCODE
// Veri Yapıları (örnek):
// Student: {id, password, GPA, current_courses: list<CourseInstance>, current_credits}
// Course: {course_id, name, credits, capacity, enrolled_count, prerequisites: list<course_id>, schedule: list<TimeSlot>}
// TimeSlot: {day, start_time, end_time}
// Selections: list<course_id> (öğrencinin seçmek istedikleri)

// Başlangıç
FUNCTION main():
    // 1. Giriş
    student_id, password <- prompt_login()
    IF NOT authenticate(student_id, password):
        show("Giriş başarısız")
        RETURN

    student <- load_student(student_id)
    available_courses <- load_all_open_courses()

    // 2. Ders listesini göster (döngü)
    display_course_list(available_courses)
    selections <- student_select_courses_loop(available_courses, student)

    // 3. Kontrolleri uygulama — her seçilen ders için
    accepted_courses <- []
    rejected_courses <- [] // tuple(course_id, reason)
    temp_total_credits <- student.current_credits

    FOR each course_id IN selections:
        course <- find_course_by_id(available_courses, course_id)

        // A) Kontenjan kontrolü
        IF course.enrolled_count >= course.capacity:
            rejected_courses.append((course_id, "Kontenjan dolu"))
            CONTINUE

        // B) Önkoşul kontrolü
        missing_prereqs <- []
        FOR each pre_id IN course.prerequisites:
            IF NOT student_has_passed(student, pre_id):
                missing_prereqs.append(pre_id)
        IF missing_prereqs IS NOT empty:
            rejected_courses.append((course_id, "Önkoşul eksik: " + join(missing_prereqs, ", ")))
            CONTINUE

        // C) Zaman çatışması kontrolü
        conflict_found <- FALSE
        FOR each existing_course IN student.current_courses + accepted_courses:
            IF schedules_conflict(course.schedule, existing_course.schedule):
                conflict_found <- TRUE
                conflict_with <- existing_course.course_id
                BREAK
        IF conflict_found:
            rejected_courses.append((course_id, "Zaman çatışması: " + conflict_with))
            CONTINUE

        // D) Kredi limiti kontrolü
        IF temp_total_credits + course.credits > 35:
            rejected_courses.append((course_id, "Kredi limiti aşılıyor (" + toString(temp_total_credits + course.credits) + ")"))
            CONTINUE

        // E) Danışman onayı gerekliliği
        IF student.GPA < 2.5:
            // Danışman onayı gerektirir — burada otomatik red veya "onay bekleniyor" mantığı olabilir
            IF NOT has_advisor_approved(student, course_id):
                rejected_courses.append((course_id, "Danışman onayı gerekli"))
                CONTINUE

        // Tüm kontrolleri geçmiş ise dersi geçici olarak kabul et
        accepted_courses.append(course)
        temp_total_credits <- temp_total_credits + course.credits

    // 4. Kayıt özeti gösterme
    show_summary(student, accepted_courses, rejected_courses, temp_total_credits)

    // 5. Kayıt onaylama
    user_confirm <- prompt_confirm("Kaydı onaylıyor musunuz?")
    IF NOT user_confirm:
        show("Kayıt iptal edildi")
        RETURN

    // 6. Kayıt işlemini kesinleştir
    FOR each course IN accepted_courses:
        enroll_student_in_course(student, course)
    show("Kayıt tamamlandı")
    show_registered_courses(student)
    RETURN

// Yardımcı Fonksiyonlar (özeti)
FUNCTION prompt_login(): ...
FUNCTION authenticate(id, pw): ...
FUNCTION load_student(id): ...
FUNCTION load_all_open_courses(): ...
FUNCTION display_course_list(courses): ...
FUNCTION student_select_courses_loop(courses, student):
    selections <- []
    WHILE True:
        choice <- prompt("Ders ekle (ders_id) veya 'bitir' yazın:")
        IF choice == 'bitir':
            BREAK
        IF not exists_course(choice): show("Geçersiz ders id"); CONTINUE
        // toggle: eğer daha önce eklenmişse çıkar
        IF choice IN selections:
            selections.remove(choice)
            show(choice + " seçimi kaldırıldı")
        ELSE:
            selections.append(choice)
            show(choice + " seçildi")
    RETURN selections

FUNCTION student_has_passed(student, course_id): ...
FUNCTION schedules_conflict(schedule1, schedule2): 
    FOR each slot1 IN schedule1:
        FOR each slot2 IN schedule2:
            IF slot1.day == slot2.day AND times_overlap(slot1.start_time, slot1.end_time, slot2.start_time, slot2.end_time):
                RETURN True
    RETURN False

FUNCTION has_advisor_approved(student, course_id): ...
FUNCTION enroll_student_in_course(student, course): 
    // artan enrolled_count ve öğrenci current_courses güncelle
    ...
FUNCTION show_summary(student, accepted, rejected, credits): ...
FUNCTION prompt_confirm(message): ...

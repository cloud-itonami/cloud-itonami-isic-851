(ns schoolops.sim
  "Demo driver -- `clojure -M:run`. Walks a clean attendance-note logging
  request through intake -> advise -> govern -> decide -> approval ->
  commit at phase 1 (assisted-logging, always approval), then re-runs
  the same op at phase 3 (supervised-auto, clean + high confidence ->
  auto-commit), then a parent-guardian-meeting-scheduling request, supply-request
  coordination, and staff-shift-proposal (all auto-commit clean at
  phase 3), then a safety-concern flag (ALWAYS escalates, at any phase
  -- approve, then commit), then HARD-hold scenarios: an unregistered
  student, a student registered but not yet verified, a proposal
  whose own `:effect` is not `:propose`, and a proposal that has
  drifted into the permanently-excluded grading/curriculum
  scope."
  (:require [langgraph.graph :as g]
            [schoolops.advisor :as advisor]
            [schoolops.store :as store]
            [schoolops.operation :as op]))

(defn- exec-op [actor tid request context]
  (g/run* actor {:request request :context context} {:thread-id tid}))

(defn- approve! [actor tid]
  (g/run* actor {:approval {:status :approved :by "school-office-coordinator-1"}} {:thread-id tid :resume? true}))

(defn -main [& _]
  (let [db (store/seed-db)
        coordinator-phase-1 {:actor-id "coord-1" :actor-role :school-office-coordinator :phase 1}
        coordinator-phase-3 {:actor-id "coord-1" :actor-role :school-office-coordinator :phase 3}
        actor (op/build db)]

    (println "== log-attendance-note student-1 (phase 1, escalates -- human approves) ==")
    (let [r (exec-op actor "t1" {:op :log-attendance-note :student-id "student-1"
                                  :patch {:status "present" :pickup "parent" :note "on time"}} coordinator-phase-1)]
      (println r)
      (println "-- human school-office coordinator approves --")
      (println (approve! actor "t1")))

    (println "\n== log-attendance-note student-1 (phase 3, clean -- auto-commits) ==")
    (println (exec-op actor "t2" {:op :log-attendance-note :student-id "student-1"
                                  :patch {:status "present" :pickup "bus" :note "on time"}} coordinator-phase-3))

    (println "\n== schedule-parent-guardian-meeting student-1 (phase 3, clean -- auto-commits) ==")
    (println (exec-op actor "t3" {:op :schedule-parent-guardian-meeting :student-id "student-1"
                                  :patch {:guardian-name "mother Yamada" :date "2026-07-20" :time "14:00"}} coordinator-phase-3))

    (println "\n== coordinate-supply-request student-1 (phase 3, clean -- auto-commits) ==")
    (println (exec-op actor "t4" {:op :coordinate-supply-request :student-id "student-1"
                                  :patch {:item "workbooks" :quantity 30 :urgency "routine"}} coordinator-phase-3))

    (println "\n== schedule-staff-shift-proposal student-1 (phase 3, clean -- auto-commits) ==")
    (println (exec-op actor "t5" {:op :schedule-staff-shift-proposal :student-id "student-1"
                                  :patch {:staff "T. Ito" :shift "morning" :date "2026-07-21"}} coordinator-phase-3))

    (println "\n== flag-safety-concern student-1 (ALWAYS escalates, even at phase 3) ==")
    (let [r (exec-op actor "t6" {:op :flag-safety-concern :student-id "student-1"
                                 :patch {:concern "unexplained bruising reported by classmate" :confidence 0.92}} coordinator-phase-3)]
      (println r)
      (println "-- human school-office coordinator reviews & approves --")
      (println (approve! actor "t6")))

    (println "\n== log-attendance-note student-99 (unregistered student -> HARD hold) ==")
    (println (exec-op actor "t7" {:op :log-attendance-note :student-id "student-99"
                                  :patch {:status "present"}} coordinator-phase-3))

    (println "\n== log-attendance-note student-3 (registered but unverified -> HARD hold) ==")
    (println (exec-op actor "t8" {:op :log-attendance-note :student-id "student-3"
                                  :patch {:status "present"}} coordinator-phase-3))

    (println "\n== schedule-parent-guardian-meeting student-1, advisor attempts direct actuation (:effect :commit) -> HARD hold ==")
    (let [actor-direct (op/build db {:advisor (reify advisor/Advisor
                                                (-advise [_ _ req]
                                                  (assoc (advisor/infer nil req) :effect :commit)))})]
      (println (exec-op actor-direct "t9" {:op :schedule-parent-guardian-meeting :student-id "student-1"
                                           :patch {:guardian-name "father" :date "2026-07-22"}} coordinator-phase-3)))

    (println "\n== log-attendance-note student-1, advisor drifts into grading/curriculum scope -> HARD hold, permanent ==")
    (println (exec-op actor "t10" {:op :log-attendance-note :student-id "student-1"
                                   :out-of-scope? true
                                   :patch {}} coordinator-phase-3))

    (println "\n== audit ledger ==")
    (doseq [f (store/ledger db)] (println f))

    (println "\n== committed coordination log ==")
    (doseq [r (store/coordination-log db)] (println r))))

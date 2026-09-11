(ns streetsweep.store
  "SSoT for the ISCO-08 9613 sweeping-and-site-cleaning-route
  scheduling/logistics coordination actor (itonami actor pattern,
  ADR-2607121000 / CLAUDE.md Actors section; README's 'Robotics
  premise' — a route scheduling/logistics coordination robot performs
  crew scheduling, cleaning-log/progress-record logging and
  cleaning-supplies/equipment procurement coordination for a
  street/site-cleaning crew under this advisor/governor pair, which
  never dispatches hardware itself, never performs sweeping/cleaning
  work itself, and never finalizes a cleaning-work-execution decision
  or a route-safety-clearance decision, and never overrides a route
  safety supervisor's judgment — those remain the route safety
  supervisor's exclusive judgment). ISCO-08 9613 (Sweepers and Related
  Labourers) is an elementary occupation (ISCO major group 9) that
  performs street/site-cleaning work outdoors, alongside or within
  roadways. Modeled closely on cloud-itonami-isco-9611's
  wastecollect.store for the outdoor/route-based elementary-occupation
  vehicle-traffic-hazard pattern, extended with a second, independent
  weather/terrain-exposure hazard-scope dimension (sweepers and
  related labourers work in public roadways and open spaces exposed
  both to moving traffic and to weather/terrain conditions that affect
  route safety and task planning).

  Domain:

    worker — a registered street/site-cleaning crew member (:worker-id,
             :name)
    route  — a registered sweeping/cleaning route {:route-id :name
             :max-supply-cost number}. `:max-supply-cost` is an
             informational registered ceiling used only to decide
             whether a `:coordinate-supply-order` proposal escalates to
             human sign-off (the governor never blocks a
             within-threshold order outright; it only decides commit
             vs. escalate).
    record — a committed operating record (a logged cleaning-log/
             progress entry, a scheduled crew operation, a flagged
             safety concern, or a coordinated supply order) — written
             ONLY via commit-record!.
    ledger — append-only audit trail, commit or hold.")

(defprotocol Store
  (worker [s worker-id])
  (route [s route-id])
  (records-of [s worker-id])
  (ledger [s])
  (register-worker! [s worker])
  (register-route! [s route])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (worker [_ worker-id] (get-in @a [:workers worker-id]))
  (route [_ route-id] (get-in @a [:routes route-id]))
  (records-of [_ worker-id] (filter #(= worker-id (:worker-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-worker! [s w]
    (swap! a assoc-in [:workers (:worker-id w)] w) s)
  (register-route! [s r]
    (swap! a assoc-in [:routes (:route-id r)] r) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:workers {} :routes {} :records [] :ledger []}
                                    seed)))))

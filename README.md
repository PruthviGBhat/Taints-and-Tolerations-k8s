KUBERNETES TAINTS & TOLERATIONS – STEP BY STEP EXECUTION

---

## STEP 1: Initial Cluster Verification

Command:
kubectl get pods
kubectl describe node ip-172-31-29-141
kubectl get nodes

Purpose:

* Verify cluster status
* Check existing taints on node

---

## STEP 2: Apply Taint (NoSchedule)

Command:
kubectl taint node ip-172-31-29-141 devops=true:NoSchedule

Purpose:

* Prevent pods from scheduling on this node unless tolerated

---

STEP 3: CASE 3 → Taint (NoSchedule) + NO Toleration
FILE: 1-pod_without_toleration.yaml
-----------------------------------

Command:
cat 1-pod_without_toleration.yaml
kubectl apply -f 1-pod_without_toleration.yaml
kubectl get pods -o wide

Expected Result:

* Pod will be in Pending state
* Node is ignored due to taint

Cleanup:
kubectl delete pod no-toleration-pod

---

STEP 4: CASE 4 → Taint (NoSchedule) + Toleration
FILE: 2-pod_with_toleration.yaml
--------------------------------

Command:
cat 2-pod_with_toleration.yaml
kubectl apply -f 2-pod_with_toleration.yaml
kubectl get pods -o wide

Expected Result:

* Pod may schedule on tainted node

Cleanup:
kubectl delete pod toleration-pod

---

## STEP 5: Add Another Taint on Different Node

Command:
kubectl taint node ip-172-31-37-64 key1=value1:NoSchedule

Re-test using:
kubectl apply -f 2-pod_with_toleration.yaml
kubectl get pods -o wide

Observation:

* Toleration must match taint key/value

---

STEP 6: CASE 6 → Taint + nodeSelector + toleration
FILE: 4-force_schedule_with_nodeselector.yaml
---------------------------------------------

Command:
cat 4-force_schedule_with_nodeselector.yaml
kubectl describe node ip-172-31-37-64 | grep -i Taints

(Edit file if needed)
nano 4-force_schedule_with_nodeselector.yaml

kubectl apply -f 4-force_schedule_with_nodeselector.yaml
kubectl get pods -o wide

Expected Result:

* Pod is guaranteed to schedule on specific tainted node

Cleanup:
kubectl delete pods --all

---

STEP 7: CASE 5 → Taint + nodeSelector WITHOUT toleration
FILE: 3-force_without_toleration.yaml
-------------------------------------

Command:
cat 3-force_without_toleration.yaml
kubectl apply -f 3-force_without_toleration.yaml
kubectl get pods -o wide

Expected Result:

* Pod remains Pending (fails to schedule)

---

## STEP 8: CASE 1 → No Taint + No nodeSelector

Command:
kubectl run testpod --image=nginx
kubectl get pods

Expected Result:

* Pod schedules on any available node

---

## STEP 9: Verify Taints on Nodes

Command:
kubectl describe node ip-172-31-37-64
kubectl describe node ip-172-31-29-141 | grep -i taints

---

## STEP 10: Remove Taint

Command:
kubectl taint node ip-172-31-37-64 key1=value1:NoSchedule-

Verify:
kubectl describe node ip-172-31-37-64 | grep -i taints

---

## STEP 11: CASE 7 → NoExecute (Eviction Scenario)

Command:
kubectl taint node ip-172-31-37-64 devops=true:NoExecute

Purpose:

* Existing pods will be evicted
* New pods will not schedule

---

STEP 12: CASE 8 → NoExecute + tolerationSeconds
FILE: 8-tolerationseconds.yaml
------------------------------

Command:
cat 8-tolerationseconds.yaml
kubectl apply -f 8-tolerationseconds.yaml
kubectl get pods -o wide -w

Expected Result:

* Pod runs temporarily
* Gets evicted after specified seconds

Cleanup:
kubectl delete pods --all

---

## STEP 13: Verify Taints Again

Command:
kubectl describe node ip-172-31-37-64 | grep -i taints
kubectl describe node ip-172-31-29-141 | grep -i taints

---

STEP 14: CASE 9 → Multiple Taints + Partial Toleration
FILE: 5-multi-toleration-fail.yaml
----------------------------------

Command:
kubectl taint node ip-172-31-37-64 random=value:NoSchedule
kubectl describe node ip-172-31-37-64

cat 5-multi-toleration-fail.yaml
kubectl apply -f 5-multi-toleration-fail.yaml
kubectl get pods -o wide

Expected Result:

* Pod remains Pending due to missing toleration for all taints

---

STEP 15: CASE 6 (FINAL SUCCESS) → Multiple Tolerations
FILE: 6-multi-toleration-success.yaml
-------------------------------------

Command:
cat 6-multi-toleration-success.yaml

(Verification step)
kubectl describe node master

Expected Result:

* Pod successfully schedules after matching all taints

---

## KEY LEARNINGS

* Taints restrict nodes
* Tolerations allow pods
* nodeSelector enforces strict placement
* NoSchedule blocks scheduling
* NoExecute evicts existing pods
* tolerationSeconds gives temporary access
* Multiple taints require multiple tolerations

---

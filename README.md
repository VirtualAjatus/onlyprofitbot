```clarity
(define-map stakeholders
  { address: principal } 
  { share: uint })

(define-data-var total-shares uint u0)

(define-public (register-stakeholder (address principal) (share uint))
  (begin
    (map-set stakeholders
      { address: address }
      { share: share })
    (ok (var-set total-shares
      (+ (var-get total-shares) share)))))

(define-public (deposit-profit)
  (let ((profit-amount (stx-get-balance)))
    (if (> profit-amount u0)
      (distribute-profit profit-amount)
      (err u"no-profit-to-distribute"))))

(define-private (distribute-profit (amount uint))
  (let ((total-shares (var-get total-shares)))
    (begin 
      (map-each stakeholders some-address stakeholder-data
        (let ((share (get share stakeholder-data))
              (share-amount (/ (* share amount) total-shares)))
          (stx-transfer? share-amount
                         (as-contract tx-sender)
                         some-address)))
      (ok u"profit-distributed"))))

;; Helper function to view a stakeholder's share
(define-read-only (get-stakeholder-share (address principal))
  (get share (map-get? stakeholders { address: address })))

;; Helper function to view total shares
(define-read-only (get-total-shares)
  (var-get total-shares))
```

### Explanation

1. **Data Structures:**
   - `stakeholders`: A map that stores stakeholder addresses and their corresponding shares.
   - `total-shares`: A data variable that keeps track of the sum of all shares.

2. **Functions:**
   - `register-stakeholder`: Registers a stakeholder and updates the total shares.
   - `deposit-profit`: Distributes the deposited amount to all the stakeholders based on their shares.
   - `distribute-profit`: A private helper function that handles the actual distribution.
   - `get-stakeholder-share`: Read-only function to retrieve a stakeholder's share.
   - `get-total-shares`: Read-only function to view the total number of shares.

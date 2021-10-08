# n-queens-vis
N Queens Visualizer for Comp105

Uses continuation passing style to solve the n-queens problem.

### Features:

* Can take any input size from 1-9 (more than enough for educational purposes)
* Allows users to step through recursive calls and failure continuations
* Faithful rendition of original uscheme solution (in console, can use N-queens in the same way as uscheme)
* Rapid notification of a solution, simple navigation through steps (and to the solution itself)

### Compromises:

* This implementation is deeply cursed, and goes against most functional programming principles.
* Lists are represented as arrays using shift, unshift, push and deep copies.
* Squares have an additional state field to help mark squares more clearly.
* There currently isn't an interactive version. I don't know if I plan on making one.

### Adapted from the following solution written in uscheme, a bridge language constructed for Comp105:
```clojure
(record square [col row])

;; Helper functions in the appendix
(define empty-board (n)
	(letrec
		[(empty-board-with-curr
			(lambda (x y)
				(if (&& (<= n x) (<= n y))
					(cons (make-square x y)'())
					(if (<= n y)
						(cons (make-square x y) 
							  (empty-board-with-curr (+ 1 x) 1))
						(cons (make-square x y) 
							  (empty-board-with-curr x (+ 1 y)))))))]
		(empty-board-with-curr 1 1)))

(define prune-squares (q safe)
	(letrec (
		[same-col? (lambda (s s') (= (square-col s) (square-col s')))]
		[same-row? (lambda (s s') (= (square-row s) (square-row s')))]
		[abs       (lambda (x)    (if (< x 0) (* -1 x) x))]
		[same-dia? (lambda (s s') (= (abs (- (square-col s) (square-col s')))
		                              (abs (- (square-row s) (square-row s')))))]
		; recursive helper function so we don't redefined the above
		; auxiliary functions every iteration
		[prune-squares' (lambda (safe')
			(if (null? safe') 
				'()
				(let ([s  (car safe')]
					  [ss (cdr safe')])
				     (if (|| (same-col? q s) (|| (same-row? q s) (same-dia? q s)))
				     	(prune-squares q ss)
				     	(cons s (prune-squares q ss))))))])
	    (prune-squares' safe)))

;; The actual n-queens solution
(define N-queens (n failure success)
    (letrec
       ([place-queens
        ;; partial solution:
        ;;    placed:        list of squares on which queens are placed
        ;;    left-to-place: number of queens left to place
        ;;    safe:         list of squares not attacked by queens
        (lambda (placed left-to-place safe f s)
        	(if (= 0 left-to-place)
        		(s placed f)
        		(if (null? safe)
        			(f)
        			(place-queens (cons (car safe) placed)
        				(- left-to-place 1)
        				(prune-squares (car safe) (cdr safe))
        				(lambda () ;; New failure continuation
        					(place-queens placed left-to-place (cdr safe) f s))
        				s))))])    ;; success continuation
       (place-queens '() n (empty-board n) failure success)))
```

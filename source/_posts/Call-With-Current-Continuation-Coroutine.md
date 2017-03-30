---
title: "Call With Current Continuation: Coroutine"
date: 2016-08-24 22:33:53
tags:

---
通过嵌套 `call/cc` 可以方便的实现协程



```scheme
 (define (coroutine routine)
   (let ((current routine)
         (status 'suspended))
     (lambda args
       (cond ((null? args) 
              (if (eq? status 'dead)
                  (error 'dead-coroutine)
                  (let ((continuation-and-value
                         (call/cc (lambda (return)
                                    (let ((returner
                                           (lambda (value)
                                             (call/cc (lambda (next)
                                                        (return (cons next value)))))))
                                      (current returner)
                                      (set! status 'dead))))))
                    (if (pair? continuation-and-value)
                        (begin (set! current (car continuation-and-value))
                               (cdr continuation-and-value))
                        continuation-and-value))))
             ((eq? (car args) 'status?) status)
             ((eq? (car args) 'dead?) (eq? status 'dead))
             ((eq? (car args) 'alive?) (not (eq? status 'dead)))
             ((eq? (car args) 'kill!) (set! status 'dead))
             (true nil)))))

```
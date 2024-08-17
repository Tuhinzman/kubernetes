1. kubectl api-resources 
   # ক্লাস্টারের সব API রিসোর্সগুলোর তালিকা দেখায়।

2. kubectl api-resources | grep pod
   # পড সম্পর্কিত রিসোর্সগুলোকে ফিল্টার করে তালিকা দেখায়।

3. kubectl explain namespace 
   # "namespace" রিসোর্স এবং এর কাঠামো সম্পর্কে বিস্তারিত তথ্য দেয়।

4. kubectl explain pod
   # "pod" রিসোর্স এবং এর কাঠামো সম্পর্কে বিস্তারিত তথ্য দেয়।

5. kubectl explain service
   # "service" রিসোর্স এবং এর কাঠামো সম্পর্কে বিস্তারিত তথ্য দেয়।

6. kubectl get nodes
   # ক্লাস্টারের সব নোডের তালিকা এবং তাদের সাধারণ স্ট্যাটাস দেখায়।

7. kubectl get nodes -o wide 
   # নোডগুলোর বিস্তারিত তথ্য দেখায়, যেমন ইন্টারনাল IP, এক্সটারনাল IP, এবং OS সম্পর্কিত তথ্য।

8. kubectl get nodes -o json
   # নোডের বিস্তারিত তথ্য JSON ফরম্যাটে আউটপুট করে।

9. kubectl get nodes -o yaml
   # নোডের বিস্তারিত তথ্য YAML ফরম্যাটে আউটপুট করে।

10. kubectl describe nodes
    # প্রতিটি নোডের স্ট্যাটাস, কনফিগারেশন, এবং কন্ডিশন সম্পর্কে বিস্তারিত তথ্য দেয়।

11. kubectl describe nodes master1 
    # "master1" নামের নোডের বিস্তারিত তথ্য দেয়।

12. kubectl describe nodes WORKER1
    # "WORKER1" নামের নোডের বিস্তারিত তথ্য দেয়।

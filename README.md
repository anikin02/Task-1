# Task - 1

Необходимо написать код генерации 5 csv файлов вида:

Категория, значение. Где категория это рандомная буква от A до D, а значение рандомное число с плавающей точкой (float к примеру)

Далее нужно параллельно обработать эти 5 файлов. Обработка заключается в поиске медианы и стандартного отклонения в рамках каждой буквы. Т.е. на выходе вы получите результат в виде:

А, медиана, отклонение

Б, медиана, отклонение

...

Д, медиана, отклонение


После этого нужно из результатов также найти медиану из медиан и стандартное отклонение из медиан. Получив такой же результат

А, медиана медиан, стандартное отклонение медиан

...


Д, то же самое


``` swift
// MARK: - S Y N C
var startTime = DispatchTime.now()
print("--- S Y N C ---")
for i in 1...5 {
  let reader = CSVReader(filePath: "/Users/anikin02/Desktop/file-\(i)")
  let categoryManager = CategoryManager(categoryValues: reader.proccesCSV())
  categoryManager.setCategoryMedianAndStandartDeviation()
  print("Sync: File-\(i)")
  print("Median")
  for (key, value) in categoryManager.categoryMedian {
    print("\(key): \(value)")
  }
  print("StandartDeviation")
  for (key, value) in categoryManager.categoryStandartDeviation {
    print("\(key): \(value)")
  }
  mediansValues.append(categoryManager.categoryMedian)
}
let endTime = DispatchTime.now()
let nanoTime = endTime.uptimeNanoseconds - startTime.uptimeNanoseconds
let timeInterval = Double(nanoTime) / 1_000_000_000
print("Execution Time: \(timeInterval) seconds")

// MARK: - A S Y N C

let group = DispatchGroup()
startTime = DispatchTime.now()
print("\n---A S Y N C ---")
for i in 1...5 {
  group.enter()
  DispatchQueue.global().async {
    let reader = CSVReader(filePath: "/Users/anikin02/Desktop/file-\(i)")
    let categoryManager = CategoryManager(categoryValues: reader.proccesCSV())
    categoryManager.setCategoryMedianAndStandartDeviation()
    
    print("\nAsync: File-\(i)")
    print("Median")
    for (key, value) in categoryManager.categoryMedian {
      print("\(key): \(value)")
    }
    print("StandartDeviation")
    for (key, value) in categoryManager.categoryStandartDeviation {
      print("\(key): \(value)")
    }
    group.leave()
  }
}
group.notify(queue: DispatchQueue.main) {
  let endTime = DispatchTime.now()
  let nanoTime = endTime.uptimeNanoseconds - startTime.uptimeNanoseconds
  let timeInterval = Double(nanoTime) / 1_000_000_000
  print("Execution Time: \(timeInterval) seconds")
  
  print("\nMedian Values")
  let medianCategory = CategoryManager(categoryValues: [:])
  medianCategory.setValues(values: mediansValues)
  medianCategory.setCategoryMedianAndStandartDeviation()
  print("Median")
  for (key, value) in medianCategory.categoryMedian {
    print("\(key): \(value)")
  }
  print("StandartDeviation")
  for (key, value) in medianCategory.categoryStandartDeviation {
    print("\(key): \(value)")
  }
}
```

``` console
--- S Y N C ---
Sync: File-1
Median
D: 500.04395
C: 500.12668
A: 499.9154
B: 499.76276
StandartDeviation
A: 288.4736
D: 288.5845
C: 288.40082
B: 288.52014
Sync: File-2
Median
A: 499.98248
D: 500.06027
C: 500.25714
B: 499.70453
StandartDeviation
B: 288.42386
C: 288.5756
A: 288.44235
D: 288.43454
Sync: File-3
Median
B: 499.73322
C: 499.73083
A: 499.87793
D: 499.82034
StandartDeviation
A: 288.3177
C: 288.52673
B: 288.55176
D: 288.4441
Sync: File-4
Median
D: 500.03238
A: 500.21774
B: 500.40472
C: 499.4904
StandartDeviation
C: 288.4462
B: 288.27335
A: 288.5143
D: 288.29077
Sync: File-5
Median
A: 499.97702
C: 499.76486
B: 500.00586
D: 500.1164
StandartDeviation
D: 288.5077
B: 288.33978
C: 288.5161
A: 288.45874
Execution Time: 424.059546 seconds

---A S Y N C ---

Async: File-2
Median
A: 499.98248
C: 500.25714
B: 499.70453
D: 500.06027
StandartDeviation
B: 288.42386
A: 288.44235
C: 288.5756
D: 288.43454

Async: File-4
Median
A: 500.21774
D: 500.03238
B: 500.40472
C: 499.4904
StandartDeviation
A: 288.5143
B: 288.27335
C: 288.4462
D: 288.29077

Async: File-5
Median
A: 499.97702
B: 500.00586
C: 499.76486
D: 500.1164
StandartDeviation
B: 288.33978
A: 288.45874
C: 288.5161
D: 288.5077

Async: File-1
Median
A: 499.9154
C: 500.12668
B: 499.76276
D: 500.04395
StandartDeviation
B: 288.52014
D: 288.5845
A: 288.4736
C: 288.40082

Async: File-3
Median
D: 499.82034
B: 499.73322
C: 499.73083
A: 499.87793
StandartDeviation
C: 288.52673
A: 288.3177
B: 288.55176
D: 288.4441
Execution Time: 155.438943541 seconds

Median Values
Median
A: 499.97702
B: 499.76276
C: 499.76486
D: 500.04395
StandartDeviation
C: 0.2793599
D: 0.10135184
A: 0.11843245
B: 0.26394948
```

Можно заметить, что асинхронные(424.059546 seconds) запросы быстрее синхронных(155.438943541 seconds) в 2,7 раз.

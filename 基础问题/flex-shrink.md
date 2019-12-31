```
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        .parent{
            display: flex;
            width: 800px;
        }
        .green{
            background: green;
            height: 100px;
            width: 300px;
            flex-shrink: 4;
        }

        .yellow{
            background: yellow;
            height: 100px;
            width: 600px;
            flex-shrink: 6;
        }
    </style>

</head>
<body>
    <div class="parent">
        <div class="green"></div>

        <div class="yellow"></div>
    </div>
</body>
```

green宽度275

yellow  525
const express = require('express');
const bodyParser = require('body-parser');
const { Sequelize, DataTypes } = require('sequelize');

const sequelize = new Sequelize('sqlite::memory:'); // Você pode trocar por MySQL/Postgres

const Product = sequelize.define('Product', {
    name: {
        type: DataTypes.STRING,
        allowNull: false
    },
    price: {
        type: DataTypes.FLOAT,
        allowNull: false
    }
});

const Tag = sequelize.define('Tag', {
    name: {
        type: DataTypes.STRING,
        allowNull: false,
        unique: true
    }
});

// Definição do relacionamento (muitos-para-muitos)
Product.belongsToMany(Tag, { through: 'ProductTags' });
Tag.belongsToMany(Product, { through: 'ProductTags' });

const app = express();
app.use(bodyParser.json());

app.post('/api/products', async (req, res) => {
    const { name, price, tags } = req.body;

    try {
        
        const product = await Product.create({ name, price });

    
        if (tags && tags.length > 0) {
            const tagList = await Promise.all(
                tags.map(tag => Tag.findOrCreate({ where: { name: tag } }))
            );
            await product.addTags(tagList.map(([tag]) => tag));
        }

        res.status(201).json(product);
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});

sequelize.sync().then(() => {
    app.listen(3000, () => {
        console.log('Server is running on port 3000');
    });
});

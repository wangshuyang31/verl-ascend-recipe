# verl-ascend-recipe
`verl-ascend-recipe` is a set of ascend examples based on [verl](https://github.com/volcengine/verl) for end-to-end RL training recipes.

## Contributing
### Recipe Folder Structure
All recipe should follow the following structure:
- `README.md`: recipe description
- `code`: recipe code
- `script`: reproducible training script

Specifically, `README.md` should contain following sections:
- Installation: which verl version is required for this recipe?
```
# release version
pip install verl==0.6.0
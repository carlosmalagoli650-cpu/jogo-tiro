
function shoot() {
  if (!playing || ended || reloading) return;

  if (ammo <= 0) {
    message('SEM MUNIÇÃO! Aperte R');
    return;
  }

  ammo--;
  updateHUD();

  raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
  raycaster.far = 1000;

  const parts = [];

  zombies.forEach(zombie => {
    zombie.traverse(part => {
      if (part.isMesh) {
        part.userData.zombie = zombie;
        parts.push(part);
      }
    });
  });

  const hits = raycaster.intersectObjects(parts, false);

  if (hits.length > 0) {
    const zombie = hits[0].object.userData.zombie;

    if (zombie && zombies.includes(zombie)) {
      scene.remove(zombie);
      zombies.splice(zombies.indexOf(zombie), 1);

      score += 100;
      updateHUD();
      message('ZUMBI ELIMINADO! +100');

      if (zombies.length === 0) finish(true);
    }
  } else {
    message('Mire no corpo do zumbi!');
  }

  gun.position.z = 0.08;
  setTimeout(() => gun.position.z = 0, 80);
}
